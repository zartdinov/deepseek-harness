# dsh web

```sh
docker build -t dsh-web:0.1.0-rc.6 .
docker run --rm -p 3080:3080 \
  -v ./workspace:/workspace \
  -v ./dsh-home:/home/node/.dsh \
  dsh-web:0.1.0-rc.6
```
