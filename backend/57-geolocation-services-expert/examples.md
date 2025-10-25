# Geolocation Services Examples

## Example: Restaurant Finder API

```javascript
const express = require('express');
const mongoose = require('mongoose');

const app = express();

// Restaurant schema with geospatial data
const restaurantSchema = new mongoose.Schema({
  name: String,
  location: {
    type: {
      type: String,
      enum: ['Point'],
      required: true,
    },
    coordinates: {
      type: [Number], // [longitude, latitude]
      required: true,
    },
  },
  address: String,
  cuisine: String,
  rating: Number,
  priceRange: String,
});

restaurantSchema.index({ location: '2dsphere' });

const Restaurant = mongoose.model('Restaurant', restaurantSchema);

// Find nearby restaurants
app.get('/api/restaurants/nearby', async (req, res) => {
  const { latitude, longitude, radius = 5000, cuisine, minRating } = req.query;

  const query = {
    location: {
      $near: {
        $geometry: {
          type: 'Point',
          coordinates: [parseFloat(longitude), parseFloat(latitude)],
        },
        $maxDistance: parseInt(radius),
      },
    },
  };

  if (cuisine) {
    query.cuisine = cuisine;
  }

  if (minRating) {
    query.rating = { $gte: parseFloat(minRating) };
  }

  const restaurants = await Restaurant.find(query).limit(20);

  // Calculate distances
  const results = restaurants.map((r) => ({
    ...r.toObject(),
    distance: calculateDistance(
      parseFloat(latitude),
      parseFloat(longitude),
      r.location.coordinates[1],
      r.location.coordinates[0]
    ),
  }));

  res.json(results);
});

// Search within delivery area (polygon)
app.post('/api/restaurants/in-area', async (req, res) => {
  const { polygon } = req.body;

  const restaurants = await Restaurant.find({
    location: {
      $geoWithin: {
        $geometry: {
          type: 'Polygon',
          coordinates: [polygon],
        },
      },
    },
  });

  res.json(restaurants);
});

function calculateDistance(lat1, lon1, lat2, lon2) {
  const R = 6371;
  const dLat = toRad(lat2 - lat1);
  const dLon = toRad(lon2 - lon1);

  const a =
    Math.sin(dLat / 2) * Math.sin(dLat / 2) +
    Math.cos(toRad(lat1)) *
      Math.cos(toRad(lat2)) *
      Math.sin(dLon / 2) *
      Math.sin(dLon / 2);

  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));

  return Math.round(R * c * 1000); // meters
}

function toRad(degrees) {
  return (degrees * Math.PI) / 180;
}
```

## Example: Ride-Sharing Driver Matching

```javascript
// Real-time driver tracking
class DriverMatcher {
  constructor() {
    this.drivers = new Map(); // driverId -> location
  }

  updateDriverLocation(driverId, latitude, longitude) {
    this.drivers.set(driverId, {
      latitude,
      longitude,
      timestamp: Date.now(),
    });
  }

  async findNearestDriver(pickupLat, pickupLng, maxDistance = 5000) {
    const available = [];

    for (const [driverId, location] of this.drivers) {
      const distance = calculateDistance(
        pickupLat,
        pickupLng,
        location.latitude,
        location.longitude
      );

      if (distance * 1000 <= maxDistance) {
        // Check if driver is available
        const driver = await db.drivers.findOne({
          _id: driverId,
          status: 'available',
        });

        if (driver) {
          available.push({
            driverId,
            distance: Math.round(distance * 1000),
            location,
            eta: this.estimateArrival(distance),
          });
        }
      }
    }

    // Sort by distance
    available.sort((a, b) => a.distance - b.distance);

    return available[0] || null;
  }

  estimateArrival(distanceKm) {
    const avgSpeed = 30; // km/h in city
    const minutes = Math.round((distanceKm / avgSpeed) * 60);
    return minutes;
  }

  removeDriver(driverId) {
    this.drivers.delete(driverId);
  }
}

// API endpoints
const matcher = new DriverMatcher();

app.post('/api/rides/request', async (req, res) => {
  const { pickupLat, pickupLng, dropoffLat, dropoffLng } = req.body;

  // Find nearest available driver
  const driver = await matcher.findNearestDriver(pickupLat, pickupLng);

  if (!driver) {
    return res.status(404).json({ error: 'No drivers available' });
  }

  // Calculate route and fare
  const route = await getRoute(
    { lat: pickupLat, lng: pickupLng },
    { lat: dropoffLat, lng: dropoffLng }
  );

  const ride = await db.rides.create({
    driverId: driver.driverId,
    pickup: {
      type: 'Point',
      coordinates: [pickupLng, pickupLat],
    },
    dropoff: {
      type: 'Point',
      coordinates: [dropoffLng, dropoffLat],
    },
    distance: route.distance,
    duration: route.duration,
    fare: calculateFare(route.distance),
    status: 'pending',
  });

  res.json({
    ride,
    driver: {
      id: driver.driverId,
      distance: driver.distance,
      eta: driver.eta,
    },
  });
});

function calculateFare(distanceMeters) {
  const baseRate = 3.0;
  const perKm = 1.5;
  const distance = distanceMeters / 1000;

  return Math.round((baseRate + distance * perKm) * 100) / 100;
}
```

## Example: Geofence-Based Notifications

```javascript
// Geofence manager
class GeofenceManager {
  async createStoreGeofence(storeId, latitude, longitude, radius = 500) {
    return await db.geofences.create({
      storeId,
      name: `Store ${storeId} Geofence`,
      center: {
        type: 'Point',
        coordinates: [longitude, latitude],
      },
      radius,
      active: true,
    });
  }

  async checkEntry(userId, latitude, longitude) {
    const userState = await db.userGeofenceState.findOne({ userId });

    const currentFences = await this.getActiveGeofences(latitude, longitude);

    const previousFenceIds = userState
      ? userState.geofences.map((g) => g.toString())
      : [];
    const currentFenceIds = currentFences.map((g) => g._id.toString());

    // Detect entries
    const entered = currentFenceIds.filter(
      (id) => !previousFenceIds.includes(id)
    );

    // Detect exits
    const exited = previousFenceIds.filter(
      (id) => !currentFenceIds.includes(id)
    );

    // Update state
    await db.userGeofenceState.upsert(
      { userId },
      {
        userId,
        geofences: currentFenceIds,
        updatedAt: new Date(),
      }
    );

    // Trigger notifications for entries
    for (const fenceId of entered) {
      const fence = currentFences.find((f) => f._id.toString() === fenceId);

      if (fence) {
        await this.sendGeofenceNotification(userId, fence, 'entered');
      }
    }

    return { entered, exited };
  }

  async getActiveGeofences(latitude, longitude) {
    const geofences = await db.geofences.find({ active: true });

    return geofences.filter((fence) => {
      const distance = calculateDistance(
        latitude,
        longitude,
        fence.center.coordinates[1],
        fence.center.coordinates[0]
      );

      return distance * 1000 <= fence.radius;
    });
  }

  async sendGeofenceNotification(userId, fence, action) {
    const store = await db.stores.findById(fence.storeId);

    await notificationService.send(userId, {
      title: `Welcome to ${store.name}!`,
      body: 'Check out our special offers today!',
      data: {
        type: 'geofence',
        action,
        storeId: store._id,
      },
    });
  }
}

// Location update endpoint
app.post('/api/location/update', async (req, res) => {
  const { latitude, longitude } = req.body;
  const userId = req.user.id;

  // Check geofences
  const manager = new GeofenceManager();
  const { entered, exited } = await manager.checkEntry(
    userId,
    latitude,
    longitude
  );

  res.json({ entered, exited });
});
```
