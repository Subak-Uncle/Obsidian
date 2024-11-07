
```
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# https://www.nginx.com/resources/wiki/start/
# https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
# https://wiki.debian.org/Nginx/DirectoryStructure
#
# In most cases, administrators will remove this file from sites-enabled/ and
# leave it as reference inside of sites-available where it will continue to be
# updated by the nginx packaging team.
#
# This file will automatically load configuration files provided by other
# applications, such as Drupal or Wordpress. These applications will be made
# available underneath a path with that package name, such as /drupal8.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##

# Default server configuration
#
        # pass PHP scripts to FastCGI server
        #
        #location ~ \.php$ {
        #       include snippets/fastcgi-php.conf;
        #
        #       # With php-fpm (or other unix sockets):
        #       fastcgi_pass unix:/run/php/php7.4-fpm.sock;
        #       # With php-cgi (or other tcp sockets):
        #       fastcgi_pass 127.0.0.1:9000;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #       deny all;
        #}

# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
#server {
#       listen 80;
#       listen [::]:80;
#
#       server_name example.com;
#
#       root /var/www/example.com;
#       index index.html;
#
#       location / {
#               try_files $uri $uri/ =404;
#       }
#}
```

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:
      auto-commit: true
    username: bootbox
    password: bootbox
    url: jdbc:mysql://127.0.0.1:3306/bootbox?allowPublicKeyRetrieval=true&useSSL=false&characterEncoding=UTF-8&useUnicode=true&serverTimezone=Asia/Seoul

  jpa:
    generate-ddl: true
    hibernate:
      ddl-auto: update
    database-platform: org.hibernate.dialect.MySQLDialect
    properties:
      hibernate:
        format_sql: true
        show_sql: true
    database: mysql
    defer-datasource-initialization: true
    show-sql: true
    open-in-view: false

  security:
    oauth2:
      client:
        registration:
          kakao:
            client-id: 042b4f3be2ec0066f4fa8267e3859964
            client-secret: L9UqNAeHob5Vcdg8Xp7mLWYAQy3POEob
            client-authentication-method: client_secret_post
            authorization-grant-type: authorization_code
            scope:
              - profile_nickname
              - profile_image
            redirect-uri: "{baseUrl}/api/oauth2/code/kakao"
            client-name: Kakao

        provider:
          kakao:
            authorization-uri: https://kauth.kakao.com/oauth/authorize
            token-uri: https://kauth.kakao.com/oauth/token
            user-info-uri: https://kapi.kakao.com/v2/user/me
            user-info-authentication-method: header
            user-name-attribute: id
  mail:
    host: smtp.gmail.com
    port: 587
    username: boootbox@gmail.com
    password: "bota jpkp hgru yqqk"
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true


jwt:
  key: sdfadsfsdfadsfasdfasdfasdfsdfnw4tn23n4on1k5n1k2n14369hn

  access:
    expiration: 1800000 # 30분 (단위 ms)

  refresh:
    expiration: 604800000 # 1주

cors:
  allowed-origins:
    - "http://localhost:3000"


```