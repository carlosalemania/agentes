# Refactoring Examples

## Example 1: Extract Method

### Before
```javascript
function renderUserProfile(user) {
  const html = '<div class="profile">';
  html += '<h1>' + user.name + '</h1>';

  if (user.avatar) {
    html += '<img src="' + user.avatar + '" alt="Avatar">';
  }

  html += '<p>' + user.bio + '</p>';

  if (user.email) {
    html += '<a href="mailto:' + user.email + '">' + user.email + '</a>';
  }

  html += '</div>';
  return html;
}
```

### After
```javascript
function renderUserProfile(user) {
  return `
    <div class="profile">
      ${renderHeader(user)}
      ${renderAvatar(user)}
      ${renderBio(user)}
      ${renderContact(user)}
    </div>
  `;
}

function renderHeader(user) {
  return `<h1>${user.name}</h1>`;
}

function renderAvatar(user) {
  return user.avatar
    ? `<img src="${user.avatar}" alt="Avatar">`
    : '';
}

function renderBio(user) {
  return `<p>${user.bio}</p>`;
}

function renderContact(user) {
  return user.email
    ? `<a href="mailto:${user.email}">${user.email}</a>`
    : '';
}
```

---

## Example 2: Replace Type Code with State Pattern

### Before
```javascript
class Order {
  constructor() {
    this.state = 'pending'; // 'pending', 'paid', 'shipped', 'delivered'
  }

  confirm() {
    if (this.state === 'pending') {
      this.state = 'paid';
    } else {
      throw new Error('Cannot confirm');
    }
  }

  ship() {
    if (this.state === 'paid') {
      this.state = 'shipped';
    } else {
      throw new Error('Cannot ship');
    }
  }

  deliver() {
    if (this.state === 'shipped') {
      this.state = 'delivered';
    } else {
      throw new Error('Cannot deliver');
    }
  }

  cancel() {
    if (this.state === 'pending' || this.state === 'paid') {
      this.state = 'cancelled';
    } else {
      throw new Error('Cannot cancel');
    }
  }
}
```

### After
```javascript
// State Pattern
class OrderState {
  confirm(order) {
    throw new Error('Cannot confirm in this state');
  }
  ship(order) {
    throw new Error('Cannot ship in this state');
  }
  deliver(order) {
    throw new Error('Cannot deliver in this state');
  }
  cancel(order) {
    throw new Error('Cannot cancel in this state');
  }
}

class PendingState extends OrderState {
  confirm(order) {
    order.setState(new PaidState());
  }
  cancel(order) {
    order.setState(new CancelledState());
  }
}

class PaidState extends OrderState {
  ship(order) {
    order.setState(new ShippedState());
  }
  cancel(order) {
    order.setState(new CancelledState());
  }
}

class ShippedState extends OrderState {
  deliver(order) {
    order.setState(new DeliveredState());
  }
}

class DeliveredState extends OrderState {}
class CancelledState extends OrderState {}

class Order {
  constructor() {
    this.state = new PendingState();
  }

  setState(state) {
    this.state = state;
  }

  confirm() {
    this.state.confirm(this);
  }

  ship() {
    this.state.ship(this);
  }

  deliver() {
    this.state.deliver(this);
  }

  cancel() {
    this.state.cancel(this);
  }
}
```

---

## Example 3: Introduce Null Object

### Before
```javascript
function renderUser(user) {
  const name = user ? user.name : 'Guest';
  const avatar = user && user.avatar ? user.avatar : '/default-avatar.png';
  const role = user ? user.role : 'visitor';

  return `
    <div>
      <img src="${avatar}" alt="${name}">
      <h2>${name}</h2>
      <span>${role}</span>
    </div>
  `;
}
```

### After
```javascript
class User {
  constructor(name, avatar, role) {
    this.name = name;
    this.avatar = avatar;
    this.role = role;
  }
}

class GuestUser extends User {
  constructor() {
    super('Guest', '/default-avatar.png', 'visitor');
  }
}

function renderUser(user = new GuestUser()) {
  return `
    <div>
      <img src="${user.avatar}" alt="${user.name}">
      <h2>${user.name}</h2>
      <span>${user.role}</span>
    </div>
  `;
}
```

---

## Example 4: Consolidate Conditional Expression

### Before
```javascript
function calculateDiscount(customer, order) {
  let discount = 0;

  if (customer.isPremium) {
    discount = 0.1;
  }

  if (order.total > 100) {
    discount += 0.05;
  }

  if (customer.purchaseCount > 10) {
    discount += 0.05;
  }

  if (customer.referralCode) {
    discount += 0.05;
  }

  return Math.min(discount, 0.3);
}
```

### After
```javascript
function calculateDiscount(customer, order) {
  const discounts = [
    isPremiumDiscount(customer),
    bulkOrderDiscount(order),
    loyaltyDiscount(customer),
    referralDiscount(customer),
  ];

  const totalDiscount = discounts.reduce((sum, d) => sum + d, 0);
  return Math.min(totalDiscount, 0.3);
}

function isPremiumDiscount(customer) {
  return customer.isPremium ? 0.1 : 0;
}

function bulkOrderDiscount(order) {
  return order.total > 100 ? 0.05 : 0;
}

function loyaltyDiscount(customer) {
  return customer.purchaseCount > 10 ? 0.05 : 0;
}

function referralDiscount(customer) {
  return customer.referralCode ? 0.05 : 0;
}
```

---

## Example 5: Replace Magic Numbers with Named Constants

### Before
```javascript
function calculateShipping(weight, distance) {
  let cost = weight * 0.5;

  if (distance > 100) {
    cost += distance * 0.1;
  } else if (distance > 50) {
    cost += distance * 0.15;
  } else {
    cost += distance * 0.2;
  }

  if (weight > 20) {
    cost *= 1.5;
  }

  return cost;
}
```

### After
```javascript
const SHIPPING_RATES = {
  BASE_WEIGHT_RATE: 0.5,
  LONG_DISTANCE_RATE: 0.1,
  MEDIUM_DISTANCE_RATE: 0.15,
  SHORT_DISTANCE_RATE: 0.2,
  HEAVY_PACKAGE_MULTIPLIER: 1.5,
};

const DISTANCE_THRESHOLDS = {
  LONG: 100,
  MEDIUM: 50,
};

const WEIGHT_THRESHOLDS = {
  HEAVY: 20,
};

function calculateShipping(weight, distance) {
  let cost = weight * SHIPPING_RATES.BASE_WEIGHT_RATE;
  cost += calculateDistanceCost(distance);

  if (isHeavyPackage(weight)) {
    cost *= SHIPPING_RATES.HEAVY_PACKAGE_MULTIPLIER;
  }

  return cost;
}

function calculateDistanceCost(distance) {
  if (distance > DISTANCE_THRESHOLDS.LONG) {
    return distance * SHIPPING_RATES.LONG_DISTANCE_RATE;
  } else if (distance > DISTANCE_THRESHOLDS.MEDIUM) {
    return distance * SHIPPING_RATES.MEDIUM_DISTANCE_RATE;
  }
  return distance * SHIPPING_RATES.SHORT_DISTANCE_RATE;
}

function isHeavyPackage(weight) {
  return weight > WEIGHT_THRESHOLDS.HEAVY;
}
```
