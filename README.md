## Overleaf example with podman compose & sandboxing

Authors: Zeus WPI (jnms)

```sh
alias p="podman"
alias pc="podman compose"

# Create podman socket readable by all
p system service --time=0 "unix://${PWD}/podman.sock" & disown
chmod a+rw podman.sock
export CONTAINER_HOST="unix://${PWD}/podman.sock"

pc up -d mongo redis
# Init the mongo cluster
pc exec mongo mongosh --eval='db.hello().primary || rs.initiate({_id: "overleaf", members:[{_id: 0, host: "mongo:27017"}]})'
pc up -d overleaf

# Create an acl to ensure the project container users can write to their volume
setfacl -dm u::rwX,g::rwX,o::rwX data/overleaf-data/compiles
```

Visit http://localhost/launchpad

Each project will spawn a (nearly stateless) container named `project-[id]-[hash of container config]`.
Old containers are cleaned up every hour.

To view all available env vars, clone the overleaf repo (https://github.com/overleaf/overleaf), and search the `services/*/config/settings.defaults.*js` files for `process.env.\w+`.
For further information, just grep all `.md` and `.js` files.
