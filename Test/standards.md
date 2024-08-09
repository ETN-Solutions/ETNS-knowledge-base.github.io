---
layout: home
title: Borrowing
parent: Performance Test Standards
---

### Performance Standards

Industry-accepted standard values for load time, latency, and processing time can vary depending on the type of application, but here are some general guidelines for web applications:

### 1. **Load Time**
   - **Ideal:** Less than 1 second
   - **Acceptable:** 1-3 seconds
   - **Suboptimal:** 3-5 seconds
   - **Poor:** More than 5 seconds

   **Note:** For e-commerce sites or sites where user engagement is critical, every additional second of load time can significantly impact user experience and conversion rates.

### 2. **Latency**
   - **Ideal:** Less than 100 ms
   - **Acceptable:** 100-300 ms
   - **Suboptimal:** 300-500 ms
   - **Poor:** More than 500 ms

   **Note:** Low latency is crucial for real-time applications, such as video conferencing, online gaming, or financial trading platforms.

### 3. **Processing Time**
   - **Ideal:** Less than 50 ms
   - **Acceptable:** 50-150 ms
   - **Suboptimal:** 150-300 ms
   - **Poor:** More than 300 ms

   **Note:** Processing time depends on the complexity of the request and server resources. For CPU-intensive tasks, higher processing times may be acceptable.

### 4. **Response Time**
   - **Ideal:** Less than 200 ms
   - **Acceptable:** 200-500 ms
   - **Suboptimal:** 500-1000 ms
   - **Poor:** More than 1 second

   **Note:** Response time is the sum of latency and processing time. It's crucial for overall user experience, especially in applications where speed is a critical factor.

### 5. **Throughput (for load testing)**
   - **Ideal:** Varies greatly depending on the application and expected load but generally, a high number of requests per second (RPS) is desired.

   **Note:** Throughput is a critical metric for understanding how many requests your application can handle under load. Balancing high throughput with low latency and processing time is key.

### Summary:
These values are general benchmarks. Actual acceptable values can vary widely based on the specific industry, application type, and user expectations. For example, a real-time financial application will have stricter requirements compared to a content-heavy website.