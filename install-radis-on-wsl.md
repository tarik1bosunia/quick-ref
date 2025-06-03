# [redis in wsl](https://redis.io/docs/latest/operate/oss_and_stack/install/install-redis/install-redis-on-windows/) 

### install redis
```sh
sudo apt-get install redis
```
```sh
### start redis server
sudo service redis-server start
```

## Connect to Redis
### Once Redis is running, you can test it by running redis-cli:
```sh
redis-cli
```
### Test the connection with the ping command:
```sh
127.0.0.1:6379> ping
PONG
```
