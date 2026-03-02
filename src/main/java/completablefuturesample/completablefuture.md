# CompletableFuture vs Future – Illustrative Example
## Flight Search Aggregator Scenario

**Scenario**:  
A backend service that queries multiple airlines in parallel to find the best flight options from Bengaluru to Goa.  
We want to:
- Query 4 airlines concurrently
- Apply per-airline timeout (4 seconds)
- Collect successful results + graceful failure messages
- Sort final offers by price
- Return everything in a clean response

**Date of discussion**: March 2026  
**Goal**: Show why CompletableFuture is usually far more practical than classic Future when dealing with multiple async operations.

### Version 1 – Classic Future (verbose and repetitive)

```java
import java.util.*;
import java.util.concurrent.*;

public FlightSearchResponse searchFlights(FlightQuery query) {
    ExecutorService pool = Executors.newFixedThreadPool(4);

    Future<List<FlightOffer>> airIndia    = pool.submit(() -> airlineClient.airIndia().search(query));
    Future<List<FlightOffer>> indigo      = pool.submit(() -> airlineClient.indigo().search(query));
    Future<List<FlightOffer>> vistara     = pool.submit(() -> airlineClient.vistara().search(query));
    Future<List<FlightOffer>> akasa       = pool.submit(() -> airlineClient.akasa().search(query));

    List<FlightOffer> allOffers = new ArrayList<>();
    List<String>      failures  = new ArrayList<>();

    try {
        allOffers.addAll(airIndia.get(4, TimeUnit.SECONDS));
    } catch (Exception e) {
        failures.add("Air India unavailable");
    }

    try {
        allOffers.addAll(indigo.get(4, TimeUnit.SECONDS));
    } catch (Exception e) {
        failures.add("IndiGo unavailable");
    }

    try {
        allOffers.addAll(vistara.get(4, TimeUnit.SECONDS));
    } catch (Exception e) {
        failures.add("Vistara unavailable");
    }

    try {
        allOffers.addAll(akasa.get(4, TimeUnit.SECONDS));
    } catch (Exception e) {
        failures.add("Akasa unavailable");
    }

    allOffers.sort(Comparator.comparing(FlightOffer::getPrice));

    return new FlightSearchResponse(allOffers, failures);
}
```
---
### Version 1 – Completable Future

```java
import java.util.*;
import java.util.concurrent.*;

public CompletableFuture<FlightSearchResponse> searchFlights(FlightQuery query) {
    // Modern choice in 2026 – virtual threads handle blocking I/O very well
    Executor executor = Executors.newVirtualThreadPerTaskExecutor();

    CompletableFuture<List<FlightOffer>> airIndia = CompletableFuture.supplyAsync(
            () -> airlineClient.airIndia().search(query), executor)
        .orTimeout(4, TimeUnit.SECONDS)
        .exceptionally(ex -> {
            log.warn("Air India failed", ex);
            return Collections.emptyList();
        });

    CompletableFuture<List<FlightOffer>> indigo = CompletableFuture.supplyAsync(
            () -> airlineClient.indigo().search(query), executor)
        .orTimeout(4, TimeUnit.SECONDS)
        .exceptionally(ex -> {
            log.warn("IndiGo failed", ex);
            return Collections.emptyList();
        });

    CompletableFuture<List<FlightOffer>> vistara = CompletableFuture.supplyAsync(
            () -> airlineClient.vistara().search(query), executor)
        .orTimeout(4, TimeUnit.SECONDS)
        .exceptionally(ex -> Collections.emptyList());

    CompletableFuture<List<FlightOffer>> akasa = CompletableFuture.supplyAsync(
            () -> airlineClient.akasa().search(query), executor)
        .orTimeout(4, TimeUnit.SECONDS)
        .exceptionally(ex -> Collections.emptyList());

    // Single wait point for all futures
    return CompletableFuture.allOf(airIndia, indigo, vistara, akasa)
        .thenApply(v -> {
            List<FlightOffer> offers = new ArrayList<>();
            List<String> errors = new ArrayList<>();

            collectResults(airIndia, offers, errors, "Air India");
            collectResults(indigo,   offers, errors, "IndiGo");
            collectResults(vistara,  offers, errors, "Vistara");
            collectResults(akasa,    offers, errors, "Akasa");

            offers.sort(Comparator.comparing(FlightOffer::getPrice));

            return new FlightSearchResponse(offers, errors);
        });
}

private void collectResults(CompletableFuture<List<FlightOffer>> cf,
                            List<FlightOffer> offers,
                            List<String> errors,
                            String airline) {
    try {
        List<FlightOffer> result = cf.join();  // safe – we waited via allOf
        if (!result.isEmpty()) {
            offers.addAll(result);
        } else {
            errors.add(airline + " had no offers");
        }
    } catch (Exception e) {
        errors.add(airline + " unavailable");
    }
}
```