# Modern C++ Expert - Examples

---

## RAII Example

```cpp
class MutexLock {
    std::mutex& mtx;
public:
    explicit MutexLock(std::mutex& m) : mtx(m) { mtx.lock(); }
    ~MutexLock() { mtx.unlock(); }
};

void thread_safe_function() {
    static std::mutex mtx;
    MutexLock lock(mtx);
    // Automatically unlocked when leaving scope
}

// Better: use std::lock_guard
void thread_safe_function() {
    static std::mutex mtx;
    std::lock_guard<std::mutex> lock(mtx);
}
```

## Modern STL

```cpp
#include <algorithm>
#include <ranges>  // C++20

std::vector<int> numbers = {1, 2, 3, 4, 5};

// ✅ Algorithm + lambda
std::transform(numbers.begin(), numbers.end(), numbers.begin(),
    [](int x) { return x * 2; });

// ✅ Ranges (C++20)
auto doubled = numbers | std::views::transform([](int x) { return x * 2; });
```

---

**Versión:** 1.0.0
