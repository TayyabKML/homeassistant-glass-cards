# Frigate, go2rtc and Scrypted without fighting over the cameras

Notes from getting four cameras into Frigate (detection and recording), HomeKit with HKSV,
and Google Home simultaneously — without stuttering video.

## The core problem: RTSP contention

Consumer PoE cameras have a hard limit on concurrent RTSP connections, and they degrade
badly before they refuse outright. Point Frigate *and* a HomeKit bridge *and* anything else
at the camera directly and you'll get stuttering in every consumer while the camera's own
mobile app looks perfect — which is exactly the symptom that sends you debugging the wrong
thing.

**Diagnostic:** stop Frigate. If the stutter in your other consumer disappears, it's
contention, not transcoding, not network, not the bridge.

## The fix: one puller, everything else restreams

Run go2rtc (embedded in Frigate) as the single client of each camera, exposing an RTSP
restream on port 8554. Every other consumer connects to go2rtc rather than to the camera.

```
Camera ──RTSP──> go2rtc (in Frigate) ──┬──> Frigate detection/recording
                                       └──> Scrypted ──> HomeKit + Google Home
```

One connection per camera, no contention, and you can add consumers freely.

## Gotchas found along the way

**Adding a camera to Scrypted's RTSP plugin works with a single stream entry only.**
Adding a sub-stream entry alongside the main one causes the camera to fail. One entry,
pointed at the go2rtc restream.

**Scrypted needs a full restart after enabling a Custom Motion Sensor extension.** The
mixin looks correctly configured in the UI but never fires until you restart. There's no
indication anything is wrong.

**A camera must be re-paired in HomeKit after adding a motion sensor** before HKSV recording
appears. The sensor is what makes HomeKit treat it as recordable.

**Keep the original camera device in Scrypted with prebuffer disabled** if you're using its
motion sensor as the event source for a camera that now streams via go2rtc.

**go2rtc's parameter whitelist is narrow.** It supports `video`, `audio`, `width`,
`height`, `rotate`, `drawtext`, `timeout`, `raw`, `input` and `hardware`. Shorthand for
`bitrate`, `rc` or `fps` is **not valid and will break the stream**. For encoder arguments
beyond the whitelist, use `#raw=`.

**Transcoding 4K for HomeKit will hitch on modest hardware.** Better to publish a
downscaled variant from go2rtc — 1080p with hardware encoding on whatever machine has an
iGPU — and point HomeKit at that. Check hardware encoder availability with `vainfo` before
assuming VAAPI will work.

**Two-way audio usually needs the vendor plugin.** If you want to talk through a doorbell,
that specific camera generally has to go through the vendor's own integration rather than
generic RTSP. It's fine to mix approaches: doorbell on the vendor plugin with Frigate
consuming its rebroadcast, everything else on the go2rtc restream.

## Where things should run

Detection is the expensive part — put Frigate wherever the Coral TPU and the disks are,
typically a NAS. The bridge (Scrypted) can live on the smaller machine. Splitting them
means a Frigate restart doesn't disturb HomeKit and vice versa.

Worth noting: query logs and recordings on spinning disks are fine; anything doing
frequent small writes is happier on an SSD. If your NAS is HDD-only, turn off file-based
query logging in whatever you're running that offers it.
