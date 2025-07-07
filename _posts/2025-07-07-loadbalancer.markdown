---
layout: post
title: " Understanding Load Balancers: Handling Traffic the Smart Way"
date: 2025-07-07 20:00:00 +12:49
comments: true
excerpt: "A deep dive into how load balancers distribute traffic, why they're essential for scalability, and the key algorithms used like Hashing and Consistent Hashing."
---


# Understanding Load Balancers: Handling Traffic the Smart Way

Load Balancer — a term we often hear when talking about systems handling heavy traffic. But what exactly is it? Why is it so important? And how does it help in distributing loads across servers?

In this blog, we’ll explore:
- What is a Load Balancer?
- Why we need it
- How it works
- The algorithms used: Hashing and Consistent Hashing  
Let’s dive in

---

## What is a Load Balancer?

A Load Balancer is a function/service/algorithm that distributes incoming traffic across multiple servers to ensure no single server is overwhelmed.

---

## Why Do We Need a Load Balancer?

Let’s imagine you built a site like Facebook. People love it and thousands are sending requests at once. Now, if all these requests hit a single server, it can crash.

### Two Types of Scaling:
1. Vertical Scaling – Increasing server power (e.g., more RAM, better CPU)
   - Expensive beyond a point
2. Horizontal Scaling – Adding more servers running the same app
   - Scalable and cost-effective

When we scale horizontally, we must use a Load Balancer to manage which server handles which request.

---

## How Does a Load Balancer Work?

The client sends a request to the Load Balancer. The Load Balancer decides which server should handle it and forwards the request accordingly.

Two connections are created:
1. Between User ↔ Load Balancer
2. Between Load Balancer ↔ Server

There are two main types of Load Balancers:
- L7 Load Balancer – Works at the Application Layer (e.g., inspects HTTP requests)
- L4 Load Balancer – Works at the Transport Layer (e.g., uses IP, TCP)

*Image Placeholder – Load Balancer in action*

---

## Algorithms Used in Load Balancing

Let’s discuss two common techniques:

---

### 1. Hashing Algorithm

#### How it works:
- Compute a hash of the user ID
- Use the modulus operator to decide which server to forward to

**Java Example**:

```java
import java.util.List;
import java.security.MessageDigest;
import java.math.BigInteger;

public class HashingLB {
    private List<String> servers = List.of("192.3.28.101", "192.3.28.102", "192.3.28.103");

    public String getServer(String userId) {
        int hash = getHash(userId);
        int index = hash % servers.size();
        return servers.get(index);
    }

    private int getHash(String input) {
        try {
            MessageDigest md = MessageDigest.getInstance("SHA-256");
            byte[] messageDigest = md.digest(input.getBytes());
            BigInteger number = new BigInteger(1, messageDigest);
            return number.intValue() & Integer.MAX_VALUE; // positive only
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args) {
        HashingLB lb = new HashingLB();
        System.out.println("Server for user123: " + lb.getServer("user123"));
    }
}
```

#### Problem:
- When a new server is added, 50% of users might be redirected to a new server, causing session loss in server-side authentication.

---

### 2. Consistent Hashing

#### How it works:
- Represent the hash space as a circle
- Map each server onto the circle using its hash
- For a request, go clockwise and pick the next server hash

*Image Placeholder – Consistent Hash Ring*

#### Benefits:
- Reduces data movement significantly when servers are added/removed.

**Java Code for Consistent Hashing**:

```java
import java.security.MessageDigest;
import java.math.BigInteger;
import java.util.NavigableMap;
import java.util.TreeMap;

public class ConsistentHashingLB {
    private final NavigableMap<Integer, String> hashCircle = new TreeMap<>();
    private final int VIRTUAL_NODES = 3;

    public void addServer(String serverIp) {
        for (int i = 0; i < VIRTUAL_NODES; i++) {
            int hash = getHash(serverIp + "#VN" + i);
            hashCircle.put(hash, serverIp);
        }
    }

    public void removeServer(String serverIp) {
        for (int i = 0; i < VIRTUAL_NODES; i++) {
            int hash = getHash(serverIp + "#VN" + i);
            hashCircle.remove(hash);
        }
    }

    public String getServer(String key) {
        if (hashCircle.isEmpty()) return null;
        int hash = getHash(key);
        if (!hashCircle.containsKey(hash)) {
            return hashCircle.ceilingEntry(hash) != null
                ? hashCircle.ceilingEntry(hash).getValue()
                : hashCircle.firstEntry().getValue();
        }
        return hashCircle.get(hash);
    }

    private int getHash(String input) {
        try {
            MessageDigest md = MessageDigest.getInstance("SHA-256");
            byte[] digest = md.digest(input.getBytes());
            BigInteger number = new BigInteger(1, digest);
            return number.intValue() & Integer.MAX_VALUE;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args) {
        ConsistentHashingLB lb = new ConsistentHashingLB();
        lb.addServer("192.3.28.101");
        lb.addServer("192.3.28.102");
        lb.addServer("192.3.28.103");

        System.out.println("Server for userABC: " + lb.getServer("userABC"));
        System.out.println("Server for userXYZ: " + lb.getServer("userXYZ"));

        lb.addServer("192.3.28.104");
        System.out.println("After adding new server:");
        System.out.println("Server for userABC: " + lb.getServer("userABC"));
        System.out.println("Server for userXYZ: " + lb.getServer("userXYZ"));
    }
}
```

---

## Conclusion

- Load balancers help us scale our applications reliably and handle huge traffic efficiently.
- Hashing is simple but suffers from massive data shift on changes.
- Consistent Hashing is smarter — minimizing impact when the server topology changes.

Next Steps:  
In the next blog, we’ll explore L4 Load Balancers, Reverse Proxies, and Health Checks used in real-world systems like Amazon and Netflix.