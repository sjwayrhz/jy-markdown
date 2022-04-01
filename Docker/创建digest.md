Latest answer

Edit suggested by OhJeez in the comments.
```
docker inspect --format='{{index .RepoDigests 0}}' $IMAGE
```
Original answer

I believe you can also get this using
```
docker inspect --format='{{.RepoDigests}}' $IMAGE
```