# Proxy

**Also known as:** Surrogate
**Category:** Structural

## Intent

Lets you provide a substitute or placeholder for another object. A proxy controls access to the original object, allowing you to perform something either before or after the request gets through to the original object.

## Problem

A massive database object wastes resources when initialized but unused. Spreading lazy-init checks across all clients creates code duplication.

## Solution

Create a Proxy class with the same interface as the original service. The proxy holds a reference to the service and can add lazy initialization, logging, access control, or caching before/after delegating to the real service.

## Variants

| Type | Purpose |
|---|---|
| Virtual Proxy | Lazy initialization (create heavy object only when needed) |
| Protection Proxy | Access control (only authorized users) |
| Remote Proxy | Local representative of a remote object (network) |
| Logging Proxy | Log each request before passing to service |
| Caching Proxy | Cache results of recurring requests |
| Smart Reference | Dismiss service when no clients reference it |

## Structure

1. **ServiceInterface** — interface shared by proxy and real service
2. **Service** — provides the actual useful business logic
3. **Proxy** — has reference to service. After completing its processing (lazy init, logging, caching, etc.), delegates to the service
4. **Client** — works with both through the same interface

## Pseudocode

```
interface ThirdPartyYouTubeLib is
    method listVideos(): VideoList
    method getVideoInfo(id): Video
    method downloadVideo(id): Stream

class ThirdPartyYouTubeClass implements ThirdPartyYouTubeLib is
    method listVideos(): VideoList is
        // Send API request to YouTube
    method getVideoInfo(id): Video is
        // Get metadata about a video
    method downloadVideo(id): Stream is
        // Download a video from YouTube

class CachedYouTubeClass implements ThirdPartyYouTubeLib is
    field service: ThirdPartyYouTubeLib
    field listCache: VideoList
    field videoCache: map of id → Video
    field needReset: bool

    constructor CachedYouTubeClass(service: ThirdPartyYouTubeLib) is
        this.service = service

    method listVideos(): VideoList is
        if (listCache == null || needReset)
            listCache = service.listVideos()
        return listCache

    method getVideoInfo(id): Video is
        if (!videoCache.has(id) || needReset)
            videoCache[id] = service.getVideoInfo(id)
        return videoCache[id]

    method downloadVideo(id): Stream is
        if (!downloadExists(id) || needReset)
            return service.downloadVideo(id)

// Client code works with proxy the same way as with the real service
class YouTubeManager is
    field service: ThirdPartyYouTubeLib

    constructor YouTubeManager(service: ThirdPartyYouTubeLib) is
        this.service = service

    method renderVideoPage(id) is
        info = service.getVideoInfo(id)
        // render the video page

    method renderListPanel() is
        list = service.listVideos()
        // render list of video thumbnails
```

## Applicability

- **Lazy initialization (virtual proxy)** — delay creating heavy service object until actually needed
- **Access control (protection proxy)** — only let specific clients use the service
- **Local execution of a remote service (remote proxy)** — proxy passes request over network
- **Logging requests (logging proxy)** — log each request before passing to service
- **Caching (caching proxy)** — cache recurring request results, manage cache lifecycle
- **Smart reference** — dismiss heavy service when no clients use it; track client references

## Pros and Cons

**Pros:**
- Control the service object without clients knowing
- Manage the lifecycle of the service object even when clients don't care
- Works even if service object isn't ready or available
- Open/Closed Principle — add new proxies without changing service or clients

**Cons:**
- Code becomes more complex with another layer
- Response from the service might be delayed

## Relations with Other Patterns

- **Adapter** provides a different interface to the wrapped object; **Proxy** provides the same interface; **Decorator** provides an enhanced interface
- **Facade** is similar to Proxy in that both buffer a complex entity. Facade hides subsystem complexity; Proxy has the same interface as its service
- **Decorator** and Proxy have similar structure but different intents: Decorator = client-managed behavior addition; Proxy = independently manages service lifecycle
