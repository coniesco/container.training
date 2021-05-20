# Tips for efficient Dockerfiles

We will see how to:

* Reduce the number of layers.

* Leverage the build cache so that builds can be faster.

---

## Reducing the number of layers

* Each line in a `Dockerfile` creates a new layer.

* Build your `Dockerfile` to take advantage of Docker's caching system.

* Combine commands by using `&&` to continue commands and `\` to wrap lines.

Note: it is frequent to build a Dockerfile line by line:

```dockerfile
RUN apt-get install thisthing
RUN apt-get install andthatthing andthatotherone
RUN apt-get install somemorestuff
```

And then refactor it trivially before shipping:

```dockerfile
RUN apt-get install thisthing andthatthing andthatotherone somemorestuff
```

---

## Avoid re-installing dependencies at each build

* Classic Dockerfile problem:

  "each time I change a line of code, all my dependencies are re-installed!"

* Solution: `COPY` dependency lists (`package.json`, `requirements.txt`, etc.)
  by themselves to avoid reinstalling unchanged dependencies every time.

---

## Example "bad" `Dockerfile`

The dependencies are reinstalled every time, because the build system does not know if `requirements.txt` has been updated.

```bash
FROM python
WORKDIR /src
COPY . .
RUN pip install -qr requirements.txt
EXPOSE 5000
CMD ["python", "app.py"]
```

---

## Fixed `Dockerfile`

Adding the dependencies as a separate step means that Docker can cache more efficiently and only install them when `requirements.txt` changes.

```bash
FROM python
COPY requirements.txt /tmp/requirements.txt
RUN pip install -qr /tmp/requirements.txt
WORKDIR /src
COPY . .
EXPOSE 5000
CMD ["python", "app.py"]
```

---

## Be careful with `chown`, `chmod`, `mv`

* Layers cannot store efficiently changes in permissions or ownership.

* Layers cannot represent efficiently when a file is moved either.

* As a result, operations like `chown`, `chown`, `mv` can be expensive.

* For instance, in the Dockerfile snippet below, each `RUN` line
  creates a layer with an entire copy of `some-file`.

  ```dockerfile
  COPY some-file .
  RUN chown www-data:www-data some-file
  RUN chmod 644 some-file
  RUN mv some-file /var/www
  ```

* How can we avoid that?

---

## Put files on the right place

* Instead of using `mv`, directly put files at the right place.

* When extracting archives (tar, zip...), merge operations in a single layer.

  Example:

  ```dockerfile
    ...
    RUN wget http://.../foo.tar.gz \
     && tar -zxf foo.tar.gz \
     && mv foo/fooctl /usr/local/bin \
     && rm -rf foo
  ...
  ```

---

## Use `COPY --chown`

* The Dockerfile instruction `COPY` can take a `--chown` parameter.

  Examples:

  ```dockerfile
  ...
  COPY --chown=1000 some-file .
  COPY --chown=1000:1000 some-file .
  COPY --chown=www-data:www-data some-file .
  ```

* The `--chown` flag can specify a user, or a user:group pair.

* The user and group can be specified as names or numbers.

* When using names, the names must exist in `/etc/passwd` or `/etc/group`.

  *(In the container, not on the host!)*

---

## Set correct permissions locally

* Instead of using `chmod`, set the right file permissions locally.

* When files are copied with `COPY`, permissions are preserved.



???

:EN:Optimizing images
:EN:- Dockerfile tips, tricks, and best practices
:EN:- Reducing build time
:EN:- Reducing image size

:FR:Optimiser ses images
:FR:- Bonnes pratiques, trucs et astuces
:FR:- Réduire le temps de build
:FR:- Réduire la taille des images
