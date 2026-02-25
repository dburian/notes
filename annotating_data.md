---
tags: [ml]
---

# Annotating data

Sometimes, as a researcher, we need data we can rely on. Small test set to test
our hypothesis. I've foud it best to use some kind of UI for quick and
streamlined annotation process. So far, I've used the following tools:

- Jupyter notebook with `ipywidgets`
- [LabelStudio](https://labelstud.io/)


## Jupyter notebook with `ipywidgets`

Best solution for annotations that require simple textual input. `ipywidgets` UI
can be quickly created and data can be viewed right away after annotation in the
notebook itself.

Pros:
- no new concepts, just programming
- annotation UI can be in git -- it can be easily used in the future

Cons:
- data retention -- restarting the notebook throws away all data
- complicated UIs or input that is not textual -- viz. interactivity


### Interacitvity

Once your UIs become too complex and require Javascript, it might be time to
look for another solution. Running Javascript within running notebook is
possible (using
[`IPython.display.Javascript`](https://ipython.readthedocs.io/en/stable/api/generated/IPython.display.html#IPython.display.Javascript)
or other solutions), but I found it cumbersome -- I didn't study it thoroughly.


## LabelStudio

LabelStudio is an opensource server that offers pre-built UIs for annotation,
which are somewhat customizable. The process goes like this:

1. Run the app using docker
  1. Use persistent data storage, so that annotatinons don't disappear once you
     kill the docker container

```bash
docker run -p 8099:8080 -v /path/to/local/persistent/dir:/label-studio/data heartexlabs/label-studio:latest
```

2. Prepare `json` definition of tasks --
   [docs](https://labelstud.io/guide/task_format)
   1. With optional predictions that can pre-fill the annotation --
      [docs](https://labelstud.io/guide/predictions)

3. Annotate inside LabelStudio

4. Export `json` annotation -- [docs](https://labelstud.io/guide/export)


Note that LabelStudio has also hosted paid versions, which offer some
[features](https://labelstud.io/guide/label_studio_compare#Feature-comparison)
that help to handle large number of annotators.

### Non-textual data

For some annotations you'd like to upload non-textual data like images or audio.
LabelStudio cannot process them in the json itself. Instead you create another
simple http server that serves the files and put url links to them into the
json.

A gpt created http server (with CORs enabled -- required) written in python:
```python
import http.server
import socketserver

PORT = 8999


class CORSRequestHandler(http.server.SimpleHTTPRequestHandler):
    def end_headers(self):
        # Allow any domain to access these files
        self.send_header("Access-Control-Allow-Origin", "*")
        # Allow common methods
        self.send_header("Access-Control-Allow-Methods", "GET, POST, OPTIONS")
        # Allow standard headers
        self.send_header("Access-Control-Allow-Headers", "X-Requested-With, Content-Type")
        super().end_headers()

    def do_OPTIONS(self):
        """Respond to browser preflight requests."""
        self.send_response(200)
        self.end_headers()


print(f"Serving files at http://localhost:{PORT}")
print("CORS is disabled (Access-Control-Allow-Origin: *)")

with socketserver.TCPServer(("", PORT), CORSRequestHandler) as httpd:
    try:
        httpd.serve_forever()
    except KeyboardInterrupt:
        print("\nServer stopped.")
        httpd.server_close()
```

Place the file inside the directory you'd like to serve and run it.
