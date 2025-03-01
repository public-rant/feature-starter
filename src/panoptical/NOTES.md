## Prompt Driven Development

We define a `--spec` option for `pytest` in [`conftest.py`](./conftest.py). This allows us to run `pytest -s panoptical_test.py --spec ${SPEC}` in [src/panoptical/install.sh](./src/panoptical/install.sh)


Really what you want to do is configure your devcontainer thusly:

__NB__ there is more than one way to configure a devcontainer

```json
{
    "mojo": {
        "image": "docker.io/library/mojo:latest",
        "features": {
            "panoptical": {
                "spec": "Update my calendar with the latest merge requests"
            }
        }
    },
    "python": {
        "image": "docker.io/library/python:latest",
        "features": {
            "panoptical": {
                "spec": "Update my calendar with the latest comments"
            }
        }
    }
}

Where `spec` is a prompt which will be sent to an LLM to generate content/code.

Since we are using python, we can hook into the python/mojo ML ecosystem. If you configure your devcontainer with `mounts`, you can "trick" calcurse-caldav.py to running as a mojo script. (This assumes you're running in a container with `mojo` available)

```json
...
"panoptical": {
    ...
    "mounts": [
        "source=/path/to/calcurse-caldav.py,target=/path/to/calcurse-caldav.mojo,type=bind,consistency=cached",
        "source=${localWorkspaceFolder}/calcurse-caldav.conf,target=/data,type=bind,consistency=cached"
    ]
}
```

## The Prompt

The prompt can be found here

```
[You are a python expert...](./panoptical_test.py)
```

### Upstream/Downstream

You need to configure an [endpoint](https://gitlab.com/public-rant/feature-starter/-/blob/main/test/panoptical/config.sample?ref_type=heads#L16-20) for `calcurse-caldav`

All your data will be pulled from there. Run multiple times with diff config to pull from multiple calendars or run container multiple times with unified mount point for notes.

Once you've pulled data using calcurse-caldav, your LLM assistant can [optimise your schedule and create new calendars](./pan_optical_test.py).


## Further work

The CI/CD extension should respond to webhooks according to rules you will need to define. E.g., if the body of a comment matches certain criteria. Or, what about hooks triggered from motion sensors in your Apple Vision Pro which prompt SORA render the scene around the corner?


## Radicale

### Why would you want this?

__calendars integrate nicely with 3rd party project management software__

I think it could be helpful to have all of the tasks your AI agents are planning expressed as a calendar. The CalDAV protocol allows attachments, and `nmh` can handle sending them.

To make this e2e, apply this patch

```diff
index d26a5bc..760b686 100644
--- a/.devcontainer/devcontainer.json
+++ b/.devcontainer/devcontainer.json
@@ -1,5 +1,7 @@
 {
-    "image": "mcr.microsoft.com/devcontainers/javascript-node:1-18-bullseye",
+    "dockerComposeFile": "docker-compose.yml",
+    "service": "devcontainer",
+    "workspaceFolder": "/workspaces/feature-starter",
     "customizations": {
         "vscode": {
             "settings": {
diff --git a/.devcontainer/docker-compose.yml b/.devcontainer/docker-compose.yml
new file mode 100644
index 0000000..8c6850f
--- /dev/null
+++ b/.devcontainer/docker-compose.yml
@@ -0,0 +1,35 @@
+version: '3.8'
+services:
+  devcontainer:
+    image: mcr.microsoft.com/devcontainers/base:ubuntu
+    volumes:
+      - ../..:/workspaces:cached
+    network_mode: service:radicale
+    command: sleep infinity
+
+  radicale:
+    image: tomsquest/docker-radicale
+    container_name: radicale
+    ports:
+      - 127.0.0.1:5232:5232
+    volumes:
+      - work:/data
+
+volumes:
+  work:
```
