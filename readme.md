## To run this example you will need:

A server running docker with a DNS name pointing to it. (If you don't already
have one setup, using [docker-machine](
https://docs.docker.com/machine/get-started-cloud/#examples) can make it easier
to setup and deploy to your server.) Your server will need ports 80, 443, and
2222 open to the public.

The following instructions  assume you're using docker-machine. This will
definitely work without it, but you'll have to do a bit more manual copying of
files. And you'll probably want to clone the repo on your server rather than
locally.

- Clone this repo
- Replace the content of `identity.pub` with your own SSH public key.
- Adjust file permissions:
```
chmod 644 identity.pub
```
- Rename the `.env.sample` file to `.env` and in it replace `replace.this.example.com` with the DNS name of your server and `replace.with.your@email.com` with your email. Letsencrypt has a pretty strict rate limit of 5 per week, so if you find yourself needing to debug things, uncomment the line `LETSENCRYPT_TEST=true`. The test certs will by considered unsecure by browsers but it should let you test the basic functionality without hitting the rate limit.
- Then to launch the docker containers:
    ```
      eval $(docker-machine env your-server-name)
      docker-compose up
    ```
- Now on your local machine you can test the tunnel with something like:
    ```
      python -m SimpleHTTPServer 3000 .
      ./start_tunnel.sh 3000 whatever.your.domain.is
    ```
- Open browser to whatever.your.domain.is and you should see an "It works!"

## The hostname
The default config for nginx will pass on the Host header so the service at the
other end of the tunnel will know the public Host. If your service is fussy and
old listens for a specific Host, you'll have to tell it to listen for your
domain, `replace.this.example.com` in this example.

Alternatively, you can tell nginx to set the host so some fixed value, like
`localhost`. **Beware** though, if your service uses the host to write
something into the response body for future requests (I'm looking at you
webpack-dev-server Hot Module Reload) then this will probably fail because the
client will send requests to the wrong Host.

Make the following changes to achieve this:

  1. create a file, `force-host.conf`, in this directory
      ```
      # force-host.conf
      proxy_set_header Host localhost;
      ```
  1. edit the `docker-compose.yml` to mount our new file into nginx
      ```yaml
      nginx-proxy:
        ...
        volumes:
          ...
          - "./force-host.conf:/etc/nginx/conf.d/force-host.conf:ro"
      ```
  1. restart the stack
      ```bash
      docker-compose up -d
      ```
