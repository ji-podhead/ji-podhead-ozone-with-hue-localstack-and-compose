
# apache ozone & hue file browser & ws localstack
- ***this tutorial is for ubuntu!***
- we will use postgres as our database
  
## aws cli for localstack
### install conda
- we need conda for pip and environments
> i followed this guide:
> https://greenwebpage.com/community/how-to-install-conda-on-ubuntu-24-04/

- create and activate a conda enironment 
- install pip

### awscli-local
```bash
pip install awscli-local
```

## install postgres and hue file browser
```yml
  network1:
    driver: bridge
    ipam:
      config:
        - subnet: 200.0.0.0/16
          gateway: 200.0.0.1
volumes:
    postgres:
services:
# # ---------- >> POSTGRES << ----------
  postgresql:
    image: postgres
    restart: always
    # set shared memory limit when using docker-compose
    shm_size: 128mb
    # or set shared memory limit when deploy via swarm stack
    #volumes:
    #  - type: tmpfs
    #    target: /dev/shm
    #    tmpfs:
    #      size: 134217728 # 128*2^20 bytes = 128Mb
    environment:
      POSTGRES_PASSWORD:  test
      POSTGRES_USER: test
      DB_NAME: test
      restart: unless-stopped
      POSTGRES_HOST_AUTH_METHOD: trust
    volumes:
      - postgres:/data/postgres
    ports:
      - "5432:5432"
    networks:
      network1:
        ipv4_address: 200.0.0.4
 hue:
    image: gethue/hue:latest
    hostname: hue
    ports:
      - "8888:8888" # Hue Web UI Port
    volumes:
      - ./hue.ini:/usr/share/hue/desktop/conf/z-hue.in
    environment:
      - HUE_SECRET_KEY=a-very-secret-key-please-change-me
      - DJANGO_ALLOWED_HOSTS=*
      - FS_DEFAULTFS=ofs://om:9862 
      # Optional: WebHDFS/HttpFS URL (OM's HTTP Port) 
      # - WEBHDFS_URL=http://om:9874/webhdfs/v1 
      - AWS_REGION=us-east-1 
      - S3_ENDPOINT_URL=http://s3g:9878
      - AWS_ACCESS_KEY_ID=YOUR_OZONE_S3_ACCESS_KEY 
      - AWS_SECRET_ACCESS_KEY=YOUR_OZONE_S3_SECRET_KEY
      - IS_YARN_ENABLED=false
    # Optional:
    # volumes:
    #   - hue_data:/usr/share/hue/desktop/conf
    networks: # Explizit Netzwerk zuweisen
      - network1
   # depends_on: 
   #   - om
   #   - s3g
```

### create hue.ini
```yml
[desktop]
[[database]]
engine=postgresql_psycopg2
host=database
port=5432
user=test
password=test
name=test
```
