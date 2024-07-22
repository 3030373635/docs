注意：环境变量传参的方式很多种，这里以文件方式为准

环境变量加载顺序：

1. Compose file：直接在docker-compose.yml中配置环境变量
2. Shell environment variables：通过shell传参
3. Environment file：env文件中的环境变量
4. Dockerfile：dockerfile中的环境变量
5. Variable is not defined：未定义

## 配置文件使用环境变量

**docker-compose.yml**

```shell
version: '3'
services:
  flower:
    build: ./layout/flower
    ports:
      - "${FLOWER_PORT}:5555"
    volumes:
      - "${PROJECT_PATH}:/opt/social"
  v2ray:
    build: ./layout/v2ray
    volumes:
      - "${PROJECT_PATH}/layout/v2ray:/etc/v2ray"
    ports:
      - "${PROXY_PORT}:8011"
      - "${FLASK_PORT}:8012"
```

**export.sh**

```shell
# flower服务端口
export FLOWER_PORT=10241

# 项目路径
export PROJECT_PATH=$(pwd)

# 代理服务端口
export PROXY_PORT=8011

# flask服务端口
export FLASK_PORT=8012
```

使用前需要将环境变量导入

```shell
(social) ╭─meng@192.168.1.7 ~/Desktop/work/pycharm_project/social
╰─➤  source export.sh 
```

## 容器使用环境变量

**docker-compose.yml**

```shell
version: '3'
services:
  flower:
    build: ./layout/flower
    env_file:
      - .env/common.env
    ports:
      - "${FLOWER_PORT}:5555"
    depends_on:
      - redis
    networks:
      - social
    volumes:
      - "${PROJECT_PATH}:/opt/social".env/common.env
```

**\.env/common.env**

```shell
REDIS_HOST=social_redis_1
REDIS_PORT=${REDIS_PORT}
REDIS_PASSWORD=social
KAFKA_HOST=${KAFKA_HOST}
KAFKA_PORT=${KAFKA_PORT}
TOPIC=SOCIALS
PROXY_HOST=social_v2ray_1
PROXY_PORT=${PROXY_PORT}
PROJECT_ENV=prod
```

env文件里面的环境变量也是支持动态传递的，本质是把文件里面的变量放在docker-compose.yml中的environment下面，等同于

```shell
version: '3'
services:
  flower:
    build: ./layout/flower
    environment: 
     REDIS_HOST=social_redis_1
      REDIS_PORT=${REDIS_PORT}
      REDIS_PASSWORD=social
      KAFKA_HOST=${KAFKA_HOST}
      KAFKA_PORT=${KAFKA_PORT}
      TOPIC=SOCIALS
      PROXY_HOST=social_v2ray_1
      PROXY_PORT=${PROXY_PORT}
      PROJECT_ENV=prod
    ports:
      - "${FLOWER_PORT}:5555"
    volumes:
      - "${PROJECT_PATH}:/opt/social".env/common.env
```

当我们执行source export.sh时，就会把参数传进来。