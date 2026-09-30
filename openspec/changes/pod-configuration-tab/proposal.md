# Proposal

## Why

From the user's smoke test of `resource-links`: "I didn't see ConfigMaps or Secrets in the pod
detail." They are there, as links on volume rows and container cards, but a link shows only a name.
Finding out what a pod is actually configured with still means following each one, one panel at a
time. FreeLens answers the same question in place: it expands every ConfigMap and Secret a pod uses
right inside the pod view, with Secret values behind a "show" button.

## What Changes

- **A new Configuration tab in pod detail.** It lists every ConfigMap and Secret the pod references,
  through volumes (plain and projected), `envFrom`/`valueFrom`, and `imagePullSecrets`. Each is a
  card headed by a link to the object, saying where the pod uses it ("volume `config` at
  `/etc/app`", "env of `web`") and showing its contents: ConfigMap keys with their values, Secret
  keys with their sizes.
- **Secret values can be revealed, one at a time.** Each Secret key has a reveal (eye) button, and
  the value shows only while revealed. This **reverses** `resource-links`' "Secret values are never
  shown" (the `object-detail` capability) on purpose, at the user's request. The object viewer's
  Secret section gets the same per-value reveal, so there is one rule, not two. The safeguards:
  - A value is fetched only when its reveal is pressed, and only that value is kept. It lives in
    memory on that one panel until hidden, the tab is left, or the panel closes.
  - It is never written to disk (dock layouts, workspace or preference files), never logged, and
    never in a window title, tab name, palette entry or notification.
  - The YAML view stays redacted exactly as it is today.
  - A panel-scoped command, Hide Secret Values, hides every revealed value at once.
- **Tab keys follow tab order.** Configuration sits after Containers, and the tab keys stay
  positional: 1 Overview, 2 Containers, 3 Configuration, 4 Volumes, 5 Events, 6 Managed Fields.
  Volumes, Events and Managed Fields each move up by one key. A `keymap.toml` override, keyed by
  command id, still applies.

## Capabilities

### Modified Capabilities
- `pod-detail`: gains the Configuration tab, and its tab keys shift.
- `object-detail`: "Secret values are never shown" becomes "Secret values are hidden until
  revealed", in both the object viewer and the new tab.

## Impact

- `app/src/k8s/resource/pod_detail/`: a new `configuration` module (projection, lazy fetch of the
  referenced objects, and the per-value reveal state) and a new `DetailSection::Configuration`.
- `app/src/k8s/resource/object_detail/`: the Secret section gains the same reveal.
- A shared `SecretValue` type (`app/src/k8s/resource/secret_value.rs`): its `Debug` and `Display`
  print a placeholder, never the value, so a value can't reach a log line by accident.
- No new dependencies. Base64 decoding uses `k8s-openapi`'s `ByteString`, which is already decoded.
