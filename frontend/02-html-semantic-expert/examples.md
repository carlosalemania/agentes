# HTML Semantic Expert - Examples

---

## Example 1: Accessible Modal Dialog

```html
<!-- ✅ GOOD - Fully accessible modal -->
<div
  id="modal"
  role="dialog"
  aria-labelledby="modal-title"
  aria-describedby="modal-description"
  aria-modal="true"
  hidden>

  <div class="modal-content">
    <header>
      <h2 id="modal-title">Confirm Action</h2>
      <button
        type="button"
        aria-label="Close dialog"
        onclick="closeModal()">
        &times;
      </button>
    </header>

    <div id="modal-description">
      <p>Are you sure you want to delete this item?</p>
    </div>

    <footer>
      <button type="button" onclick="confirm()">Confirm</button>
      <button type="button" onclick="closeModal()">Cancel</button>
    </footer>
  </div>
</div>

<script>
function openModal() {
  const modal = document.getElementById('modal');
  modal.hidden = false;
  modal.querySelector('button').focus(); // Focus trap
}

function closeModal() {
  const modal = document.getElementById('modal');
  modal.hidden = true;
  document.getElementById('trigger-button').focus(); // Return focus
}
</script>
```

## Example 2: Accessible Form with Validation

```html
<form novalidate>
  <div class="form-group">
    <label for="username">
      Username <span aria-label="required">*</span>
    </label>
    <input
      type="text"
      id="username"
      name="username"
      required
      aria-required="true"
      aria-invalid="false"
      aria-describedby="username-error username-hint">
    <span id="username-hint" class="hint">
      Must be 3-20 characters
    </span>
    <span id="username-error" class="error" role="alert" hidden>
      Username is required
    </span>
  </div>

  <div class="form-group">
    <label for="password">Password</label>
    <input
      type="password"
      id="password"
      name="password"
      required
      aria-required="true"
      aria-describedby="password-requirements">
    <ul id="password-requirements">
      <li>At least 8 characters</li>
      <li>One uppercase letter</li>
      <li>One number</li>
    </ul>
  </div>

  <button type="submit">Create Account</button>
</form>
```

## Example 3: Article with Structured Data

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <title>Understanding Semantic HTML | Developer Blog</title>
  <meta name="description" content="Learn how to write semantic HTML for better accessibility and SEO">

  <!-- Open Graph -->
  <meta property="og:title" content="Understanding Semantic HTML">
  <meta property="og:description" content="Learn semantic HTML best practices">
  <meta property="og:image" content="https://example.com/images/og-semantic-html.jpg">
  <meta property="og:type" content="article">
  <meta property="og:url" content="https://example.com/blog/semantic-html">

  <!-- Canonical -->
  <link rel="canonical" href="https://example.com/blog/semantic-html">

  <!-- JSON-LD Structured Data -->
  <script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "BlogPosting",
    "headline": "Understanding Semantic HTML",
    "author": {
      "@type": "Person",
      "name": "Jane Developer"
    },
    "datePublished": "2025-01-20T09:00:00Z",
    "dateModified": "2025-01-20T09:00:00Z",
    "image": "https://example.com/images/semantic-html.jpg",
    "publisher": {
      "@type": "Organization",
      "name": "Developer Blog",
      "logo": {
        "@type": "ImageObject",
        "url": "https://example.com/logo.png"
      }
    }
  }
  </script>
</head>
<body>
  <a href="#main-content" class="skip-link">Skip to main content</a>

  <header>
    <h1>Developer Blog</h1>
    <nav aria-label="Main navigation">
      <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/blog" aria-current="page">Blog</a></li>
        <li><a href="/about">About</a></li>
      </ul>
    </nav>
  </header>

  <main id="main-content">
    <article>
      <header>
        <h1>Understanding Semantic HTML</h1>
        <p>
          Published on <time datetime="2025-01-20">January 20, 2025</time>
          by <span>Jane Developer</span>
        </p>
      </header>

      <section>
        <h2>What is Semantic HTML?</h2>
        <p>Semantic HTML uses meaningful tags...</p>
      </section>

      <section>
        <h2>Benefits</h2>
        <ul>
          <li>Better accessibility</li>
          <li>Improved SEO</li>
          <li>Easier maintenance</li>
        </ul>
      </section>
    </article>
  </main>

  <footer>
    <p>&copy; 2025 Developer Blog. All rights reserved.</p>
  </footer>
</body>
</html>
```

---

**Versión:** 1.0.0
