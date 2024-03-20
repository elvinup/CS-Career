This [repo](https://github.com/jriddle-sf/generate-xkcd-password) is great for creating a simple password with a container.

- clone the repo
- cd into the folder and build the image
```
docker build -t --tag xkcd_password .
```
- run the container off the image
```
docker run --rm xkcd_password
```

It will spit out a password and conveniently its bcrypt hash like this:

```json
{"password": "Icon-Frill-Hypnosis-Operation-Casualty-Gauze", "hash": "$6$lAC7rfAEWfvf4wnP$kk87l7fYViju2a12QXkoP4I8FIBqwroPXrT3c55QU/YyfYMIj5tYEJ1lRx2BiqqqOUlHroAmnoFS2GpjTKyk50"}
```