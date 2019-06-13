# 下载镜像

```
docker pull arangodb
```
#  启动容器

```

mkdir /tmp/arangodb
docker run -e ARANGO_ROOT_PASSWORD=hushiwei -p 8529:8529 -d \
          -v /tmp/arangodb:/var/lib/arangodb3 \
          arangodb
```

locaalhost:8529