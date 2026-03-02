# Uber Frontend Interview Preparation Guide

> A comprehensive guide covering key JavaScript topics, concepts, and resources
> specifically curated for Uber Frontend Engineer interviews.

---

## 📋 Table of Contents

1. [What Uber Looks For](#what-uber-looks-for)
2. [Key Topics](#key-topics)
   - [JavaScript Core](#1--javascript-core-deep-level)
   - [Async JavaScript](#2--async-javascript)
   - [Performance Optimization](#3--performance-optimization)
   - [DOM & Browser APIs](#4--dom--browser-apis)
   - [Frontend System Design](#5--frontend-system-design)
   - [React Deep Dive](#6--react-deep-dive)
   - [DSA Frontend Focused](#7--data-structures--algorithms)
   - [Security & Reliability](#8--security--reliability)
   - [Networking & APIs](#9--networking--apis)
3. [Interview Process](#uber-interview-process)
4. [Resources](#resources)
   - [JavaScript Core Resources](#1--javascript-core-resources)
   - [React Resources](#2--react-resources)
   - [System Design Resources](#3--frontend-system-design-resources)
   - [Coding Practice](#4--coding-practice)
   - [Performance Resources](#5--performance-optimization-resources)
   - [Networking Resources](#6--networking--browser-resources)
   - [Behavioral Resources](#7--behavioral-preparation)
   - [Mock Interviews](#8--mock-interviews)
   - [Newsletters & Blogs](#9--newsletters--blogs)
   - [YouTube Channels](#10--youtube-channels)
5. [Study Plan](#study-plan)
6. [Budget Summary](#budget-summary)

---

## 🎯 What Uber Looks For

- Strong **CS fundamentals** applied to frontend
- **Performance-conscious** coding
- **Scalable** component & architecture design
- **Real-world** problem solving
- Clean, maintainable code

---

## Key Topics

---

### 1. ⚡ JavaScript Core (Deep Level)

#### Must Know:
- **Event Loop** in depth
  - Microtask vs Macrotask queue
  - `Promise` vs `setTimeout` execution order
  - `requestAnimationFrame` timing
- **Memory management**
  - Garbage collection
  - Memory leaks (detached DOM nodes, closures, timers)
  - WeakMap/WeakSet use cases
- **Closures** — practical use cases
- **Prototype chain** & inheritance
- **Coercion edge cases**

#### Likely Coding Questions:

```js
// Implement your own Promise
class MyPromise {
  constructor(executor) { ... }
  then(onFulfilled, onRejected) { ... }
  catch(onRejected) { ... }
  static all(promises) { ... }
  static race(promises) { ... }
}

// Implement Promise.all from scratch
Promise.myAll = function(promises) {
  return new Promise((resolve, reject) => {
    let results = [];
    let count = 0;
    promises.forEach((p, i) => {
      Promise.resolve(p).then(val => {
        results[i] = val;
        if (++count === promises.length) resolve(results);
      }).catch(reject);
    });
  });
};
```
### 2. 🔄 Async JavaScript (Critical for Uber)
```js
> Uber deals with **real-time data** (maps, pricing, trips) — async mastery is essential.

#### Key Topics:
- **Promise chaining** & error handling
- **Async/Await** patterns
- **Race conditions** & how to handle them
- **Request cancellation** (AbortController)
- **Retry logic** with exponential backoff
- **Polling vs WebSockets vs SSE**
```
#### Likely Coding Questions:

```js
// Implement retry with exponential backoff
async function fetchWithRetry(url, retries = 3, delay = 1000) {
  try {
    return await fetch(url);
  } catch (err) {
    if (retries === 0) throw err;
    await new Promise(res => setTimeout(res, delay));
    return fetchWithRetry(url, retries - 1, delay * 2);
  }
}

// Implement request cancellation
function fetchWithCancel(url) {
  const controller = new AbortController();
  const promise = fetch(url, { signal: controller.signal });
  return { promise, cancel: () => controller.abort() };
}

// Execute N promises with concurrency limit
async function runWithConcurrency(tasks, limit) {
  const results = [];
  const executing = [];
  for (const task of tasks) {
    const p = Promise.resolve().then(task);
    results.push(p);
    if (limit <= tasks.length) {
      const e = p.then(() => executing.splice(executing.indexOf(e), 1));
      executing.push(e);
      if (executing.length >= limit) await Promise.race(executing);
    }
  }
  return Promise.all(results);
}
```

---

### 3. 🏎️ Performance Optimization (Very Important at Uber)

> Uber's map UI is extremely performance-sensitive.

#### Key Topics:
- **Debounce & Throttle** (implement from scratch)
- **Virtualization** (rendering large lists — 10k+ items)
- **Lazy loading** (images, components, routes)
- **Code splitting** & dynamic imports
- **Critical rendering path**
- **Web Vitals** (LCP, FID, CLS, INP)
- **Reflow & Repaint** — how to minimize
- **requestAnimationFrame** for animations
- **Web Workers** for heavy computation

#### Likely Coding Questions:

```js
// Implement Debounce
function debounce(fn, delay) {
  let timer;
  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

// Implement Throttle
function throttle(fn, limit) {
  let lastCall = 0;
  return function(...args) {
    const now = Date.now();
    if (now - lastCall >= limit) {
      lastCall = now;
      return fn.apply(this, args);
    }
  };
}

// Implement Memoization with cache limit
function memoize(fn, cacheLimit = 100) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    if (cache.size >= cacheLimit) {
      cache.delete(cache.keys().next().value);
    }
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}
```

---

### 4. 🗺️ DOM & Browser APIs (Map-Heavy UI)

#### Key Topics:
- **Event delegation** (handling 1000s of markers efficiently)
- **Intersection Observer** (lazy loading map elements)
- **MutationObserver** (watching DOM changes)
- **ResizeObserver**
- **Canvas / WebGL** basics (for map rendering)
- **requestAnimationFrame** for smooth animations
- **Shadow DOM** basics

#### Likely Coding Questions:

```js
// Implement Event Delegation
function delegate(parent, selector, event, handler) {
  parent.addEventListener(event, function(e) {
    if (e.target.matches(selector)) {
      handler.call(e.target, e);
    }
  });
}

// Implement Virtual Scroll (simplified)
class VirtualScroller {
  constructor({ container, itemHeight, totalItems, renderItem }) {
    this.itemHeight = itemHeight;
    this.totalItems = totalItems;
    this.renderItem = renderItem;
    this.container = container;
    this.container.addEventListener('scroll', () => this.render());
    this.render();
  }

  render() {
    const scrollTop = this.container.scrollTop;
    const viewportHeight = this.container.clientHeight;
    const startIdx = Math.floor(scrollTop / this.itemHeight);
    const endIdx = Math.min(
      startIdx + Math.ceil(viewportHeight / this.itemHeight),
      this.totalItems
    );
    this.container.innerHTML = '';
    for (let i = startIdx; i < endIdx; i++) {
      this.container.appendChild(this.renderItem(i));
    }
  }
}
```

---

### 5. 🏗️ Frontend System Design (Round by Itself)

> Uber **heavily tests** system design for senior roles.

#### Common Design Questions:
- Design **Google Maps UI** / Uber's trip tracking
- Design a **real-time ride status** component
- Design an **autocomplete search** at scale
- Design a **notification system**
- Design **infinite scroll feed**

#### Framework to Answer:

```
1. Clarify requirements (functional & non-functional)
2. Component architecture
3. State management approach
4. API design (REST/WebSocket/GraphQL)
5. Performance considerations
6. Accessibility
7. Error handling & edge cases
8. Scalability
```

#### Key Concepts to Know:
- **WebSockets** for real-time location updates
- **State management** (Redux, Zustand, Context)
- **Optimistic UI updates**
- **Offline support** (Service Workers, IndexedDB)
- **CDN & caching strategies**
- **Micro-frontend architecture**

---

### 6. ⚛️ React (Deep Level)

> Uber uses React extensively.

#### Key Topics:
- **Reconciliation** & Virtual DOM diffing
- **Fiber architecture** basics
- **useMemo, useCallback, React.memo** — when & why
- **useRef** for DOM & mutable values
- **Custom hooks** — design patterns
- **Context API** performance pitfalls
- **Suspense & Concurrent Mode**
- **Error Boundaries**
- **Code splitting** with React.lazy

#### Likely Coding Questions:

```jsx
// Implement useDebounce hook
function useDebounce(value, delay) {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  return debounced;
}

// Implement usePrevious hook
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => { ref.current = value; }, [value]);
  return ref.current;
}

// Implement useIntersectionObserver
function useIntersectionObserver(ref, options) {
  const [isVisible, setIsVisible] = useState(false);
  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => setIsVisible(entry.isIntersecting),
      options
    );
    if (ref.current) observer.observe(ref.current);
    return () => observer.disconnect();
  }, [ref, options]);
  return isVisible;
}
```

---

### 7. 🧮 Data Structures & Algorithms (Frontend Focused)

> Uber does ask **DSA** but with a frontend twist.

#### Key Topics:

| DSA Topic | Frontend Context |
|-----------|-----------------|
| **Queues** | Event queue, task scheduling |
| **Trees** | DOM tree traversal |
| **Graphs** | Route mapping, navigation |
| **Hash Maps** | Caching, memoization |
| **Linked Lists** | Undo/redo history |
| **Heap/Priority Queue** | Notification priority |

#### Likely Coding Questions:

```js
// LRU Cache (very common at Uber)
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
  }
  get(key) {
    if (!this.cache.has(key)) return -1;
    const val = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, val);
    return val;
  }
  put(key, value) {
    if (this.cache.has(key)) this.cache.delete(key);
    if (this.cache.size >= this.capacity) {
      this.cache.delete(this.cache.keys().next().value);
    }
    this.cache.set(key, value);
  }
}

// Flatten nested object
function flattenObject(obj, prefix = '') {
  return Object.keys(obj).reduce((acc, key) => {
    const newKey = prefix ? `${prefix}.${key}` : key;
    if (typeof obj[key] === 'object' && !Array.isArray(obj[key])) {
      Object.assign(acc, flattenObject(obj[key], newKey));
    } else {
      acc[newKey] = obj[key];
    }
    return acc;
  }, {});
}
```

---

### 8. 🔒 Security & Reliability

#### Key Topics:
- **XSS** (Cross-Site Scripting) prevention
- **CSRF** protection
- **Content Security Policy (CSP)**
- **CORS** handling
- **Input sanitization**
- **Rate limiting** on frontend

---

### 9. 🌐 Networking & APIs

#### Key Topics:
- **HTTP/2 vs HTTP/3**
- **REST vs GraphQL vs WebSockets**
- **Caching** (Cache-Control, ETags, Service Workers)
- **Request batching & deduplication**
- **gRPC basics**

---

## 🔥 Top 10 Topics Priority

| Priority | Topic |
|----------|-------|
| 🔴 Critical | Event Loop & Async Patterns |
| 🔴 Critical | Frontend System Design |
| 🔴 Critical | Performance Optimization |
| 🔴 Critical | React internals & Hooks |
| 🟠 High | LRU Cache & DSA |
| 🟠 High | WebSockets & Real-time |
| 🟠 High | Debounce/Throttle/Memoize |
| 🟡 Medium | Browser APIs & DOM |
| 🟡 Medium | Security (XSS/CSRF) |
| 🟡 Medium | Networking & Caching |

---

## 📅 Uber Interview Process

```
Round 1: JavaScript Fundamentals + Coding
Round 2: React / Frontend Framework Deep Dive
Round 3: Frontend System Design
Round 4: DSA (with frontend context)
Round 5: Behavioral (Leadership Principles)
```

---

## Resources

---

### 1. 📚 JavaScript Core Resources

#### Free
| Resource | URL | Why Use It |
|----------|-----|------------|
| javascript.info | https://javascript.info | Best JS resource online |
| MDN Web Docs | https://developer.mozilla.org | Official JS/Web API reference |
| You Don't Know JS | https://github.com/getify/You-Dont-Know-JS | Deep JS internals — Free on GitHub |

#### YDKJS Must Read Books:
- *Scope & Closures*
- *this & Object Prototypes*
- *Async & Performance*

#### Paid
| Resource | URL | Cost | Why Use It |
|----------|-----|------|------------|
| Frontend Masters | https://frontendmasters.com | ~$39/month | JS: Hard Parts, Deep JS Foundations |

---

### 2. ⚛️ React Resources

#### Free
| Resource | URL | Why Use It |
|----------|-----|------------|
| React Official Docs | https://react.dev | Hooks, Reconciliation, Performance |
| Kent C. Dodds Blog | https://kentcdodds.com/blog | Best React patterns & hooks |
| React Fiber Architecture | https://github.com/acdlite/react-fiber-architecture | React internals |

#### Kent C. Dodds Must Read Posts:
- *When to useMemo and useCallback*
- *How React uses Closures*
- *Application State Management*

#### Paid
| Resource | URL | Cost | Why Use It |
|----------|-----|------|------------|
| Epic React | https://epicreact.dev | ~$300 one time | Most comprehensive React course |

---

### 3. 🏗️ Frontend System Design Resources

#### Free
| Resource | URL | Why Use It |
|----------|-----|------------|
| Chirag Goel YouTube | https://www.youtube.com/c/engineercodedotcom | Autocomplete, Infinite Scroll designs |
| GreatFrontEnd System Design | https://www.greatfrontend.com/system-design | Real frontend system design questions |
| Patterns.dev | https://www.patterns.dev | Design & performance patterns |
| Web.dev by Google | https://web.dev | Performance, Core Web Vitals, PWA |

#### Paid
| Resource | URL | Cost | Why Use It |
|----------|-----|------|------------|
| GreatFrontEnd Full | https://www.greatfrontend.com | ~$99 | Best targeted frontend interview prep |
| Frontend Masters System Design | https://frontendmasters.com | ~$39/month | System design for frontend engineers |

---

### 4. 💻 Coding Practice

#### Platforms

| Platform | URL | Cost | Best For |
|----------|-----|------|----------|
| GreatFrontEnd | https://www.greatfrontend.com | Free + ~$99 | Frontend-specific coding ⭐ |
| LeetCode | https://leetcode.com | Free + $35/month | DSA, Uber tagged questions |
| Frontend Mentor | https://www.frontendmentor.io | Free | Real UI challenges |
| JS Challenger | https://www.jschallenger.com | Free | Pure JS coding challenges |

#### Uber-Specific LeetCode Questions

| # | Problem | Topic |
|---|---------|-------|
| 146 | LRU Cache | HashMap + LinkedList |
| 20 | Valid Parentheses | Stack |
| 56 | Merge Intervals | Arrays |
| 42 | Trapping Rain Water | Two Pointer |
| 79 | Word Search | Backtracking |
| 207 | Course Schedule | Graph |
| 295 | Find Median from Data Stream | Heap |

---

### 5. ⚡ Performance Optimization Resources

#### Free
| Resource | URL | Why Use It |
|----------|-----|------------|
| web.dev/performance | https://web.dev/performance | Google's official perf guide |
| Chrome DevTools Docs | https://developer.chrome.com/docs/devtools | Profiling & memory leak detection |
| 3perf.com | https://3perf.com/talks/web-perf-101 | Web performance 101 |
| Chrome Developers YouTube | https://www.youtube.com/@ChromeDevs | Performance talks & tutorials |

---

### 6. 🌐 Networking & Browser Resources

#### Free
| Resource | URL | Why Use It |
|----------|-----|------------|
| How Browsers Work | https://web.dev/articles/howbrowserswork | Classic deep dive |
| HTTP/2 Explained | https://http2-explained.haxx.se | Free ebook, networking fundamentals |
| High Performance Browser Networking | https://hpbn.co | Free online book by Ilya Grigorik |

---

### 7. 🧠 Behavioral Preparation

| Resource | URL | Cost | Why Use It |
|----------|-----|------|------------|
| Uber Engineering Blog | https://www.uber.com/blog/engineering | Free | Understand Uber's tech challenges |
| STAR Method Guide | https://www.themuse.com/advice/star-interview-method | Free | Structure behavioral answers |
| Grokking Behavioral Interview | https://www.educative.io/courses/grokking-the-behavioral-interview | ~$79 | Comprehensive behavioral prep |

---

### 8. 🎯 Mock Interviews

| Platform | URL | Cost | Why Use It |
|----------|-----|------|------------|
| Interviewing.io | https://interviewing.io | Free + Paid sessions | Real engineers from top companies |
| Pramp | https://www.pramp.com | Free | Peer-to-peer mock interviews |
| Meetapro | https://meetapro.com | ~$100-200/session | Ex-Uber engineers mock interviews |

---

### 9. 📰 Newsletters & Blogs

| Resource | URL | Focus |
|----------|-----|-------|
| bytes.dev | https://bytes.dev | JS News |
| This Week in React | https://thisweekinreact.com | React News |
| CSS-Tricks | https://css-tricks.com | Frontend Tips |
| Smashing Magazine | https://smashingmagazine.com | Frontend In Depth |
| overreacted.io | https://overreacted.io | Dan Abramov's Blog |
| Josh W Comeau | https://joshwcomeau.com | CSS/React/JS |

---

### 10. 📺 YouTube Channels

| Channel | URL | Best For |
|---------|-----|----------|
| Akshay Saini | https://www.youtube.com/@akshaymarch7 | JS Fundamentals (Namaste JS) |
| Jack Herrington | https://www.youtube.com/@jherr | React Advanced |
| Theo - t3.gg | https://www.youtube.com/@t3dotgg | Modern Frontend |
| Kevin Powell | https://www.youtube.com/@KevinPowell | CSS Deep Dive |
| Fireship | https://www.youtube.com/@Fireship | Quick Concepts |
| Web Dev Simplified | https://www.youtube.com/@WebDevSimplified | React & JS |
| The Primeagen | https://www.youtube.com/@ThePrimeagen | DSA & Performance |

---

## 📅 Study Plan

### 8-Week Preparation Schedule

#### Week 1-2: JavaScript Core
- [ ] Read YDKJS: Scope & Closures
- [ ] Read javascript.info async section
- [ ] Practice 10 JS challenges/day on GreatFrontEnd
- [ ] Implement: Promise, debounce, throttle from scratch
- [ ] Study: Event Loop, Closures, Prototype chain

#### Week 3-4: React Deep Dive
- [ ] Read all React official docs (hooks section)
- [ ] Read Kent C. Dodds must-read blog posts
- [ ] Build 3-4 complex components from scratch
- [ ] Study: React reconciliation & Fiber architecture
- [ ] Implement: Custom hooks (useDebounce, usePrevious, useIntersectionObserver)

#### Week 5: Performance & Browser
- [ ] Complete web.dev performance guide
- [ ] Deep dive into Core Web Vitals
- [ ] Practice Chrome DevTools profiling
- [ ] Implement: Virtual scroll, lazy loading
- [ ] Study: requestAnimationFrame, Web Workers

#### Week 6: Frontend System Design
- [ ] Read GreatFrontEnd system design guide
- [ ] Watch Chirag Goel's YouTube series
- [ ] Practice designing: autocomplete, maps UI, infinite scroll
- [ ] Study: WebSockets, real-time patterns
- [ ] Study: Micro-frontend architecture

#### Week 7: DSA Practice
- [ ] Solve all Uber-tagged LeetCode questions
- [ ] Focus: LRU Cache, Graph problems
- [ ] Solve 2-3 problems/day
- [ ] Review time/space complexity for each solution
- [ ] Practice explaining solutions out loud

#### Week 8: Mock Interviews & Review
- [ ] Complete 2-3 mock interviews on Pramp/Interviewing.io
- [ ] Review all weak areas identified
- [ ] Read Uber Engineering blog (latest 5-10 posts)
- [ ] Prepare 5-7 behavioral stories using STAR method
- [ ] Review all code snippets and implementations

---

## 💰 Budget Summary

| Budget | Resources Included |
|--------|-------------------|
| **$0** | javascript.info, YDKJS, MDN, LeetCode free, Pramp, web.dev, All YouTube channels, Newsletters |
| **~$100** | Above + GreatFrontEnd full access + LeetCode Premium |
| **~$200** | Above + Frontend Masters (1 month) |
| **~$500+** | Above + Epic React + Paid mock interviews |

---

## 💡 Pro Tips

> - Start with **javascript.info + YDKJS** for strong JS fundamentals
> - **GreatFrontEnd** is the most targeted resource for frontend interviews
> - Read **Uber's Engineering Blog** before your interview to show genuine interest
> - Do **at least 2-3 mock interviews** before the real thing
> - Always think about **scale and performance** — mention trade-offs in every answer
> - Quality over quantity — **understand deeply** rather than memorizing solutions
> - For system design: always start by **clarifying requirements** before jumping to solutions

---

*Last Updated: 2026*
