# letsencrypt-alibaba-docker
The docker image for letsencrypt-alibaba repository.

[Let's Encrypt](https://letsencrypt.org/getting-started/) can issue a certificate for your website’s domain. 

If you wish to generate a wildcard certificate, Let's Encrypt will ask you to configure a special subdomain under your domain name and set it to a TXT type value for Let's Encrypt to determine that you do have the rights to use the domain. For example, generate a certificate for the domain *.example.com so that you can use the same certificate and support many subdomains, which is very useful in some business scenarios.

By default, you need to log in to your domain name service provider and set the value of the specified subdomain of type TXT as required and wait for an undetermined length of time before proceeding to the next step. Otherwise, you are most likely to face a failure. This library encapsulates the entire authentication process, including setting the specified value to your domain name service provider, automatically determining whether the set value has taken effect and triggering Let's Encrypt's challenge authentication, automatically registering users, etc.

# Usage

## 1. Configure the .env file

Variable | Mandatory | Default | Description | Example
--- | --- | --- | --- | ---
`DOMAIN` | YES | No Default Value | The top-level domain you want to get a wildcard certificate | futlabs.com
`ACCESS_KEY` | YES | No Default Value | The API access key provided by your domain provider | 
`ACCESS_SECRET` | YES | No Default Value | The API access secret provided by your domain provider | 
`STAGE` | NO | TEST | TEST or PROD,If you want to get a real certificate , please set to PROD. refer to [Staging Environment](https://letsencrypt.org/docs/staging-environment/) for more information | TEST
`DNS_PROVIDER` | NO | alibaba | Only _alibaba_ domain provider is currently supported  | alibaba

## 2. Run

```shell

$ docker-compose up

```

You will see something output in the console like this:

```shell

Creating letsencrypt ... done
Attaching to letsencrypt
letsencrypt    |
letsencrypt    |   .   ____          _            __ _ _
letsencrypt    |  /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
letsencrypt    | ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
letsencrypt    |  \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
letsencrypt    |   '  |____| .__|_| |_|_| |_\__, | / / / /
letsencrypt    |  =========|_|==============|___/=/_/_/_/
letsencrypt    |  :: Spring Boot ::                (v2.5.2)
letsencrypt    |
letsencrypt    | 2021-09-14 06:22:54.848  INFO 1 --- [           main] c.f.letsencrypt.WildcardApplication      : Starting WildcardApplication v0.0.2-SNAPSHOT using Java 16.0.1 on b14574bead2d with PID 1 (/app/letsencrypt-0.0.2-SNAPSHOT.jar started by root in /app)
letsencrypt    | 2021-09-14 06:22:54.858  INFO 1 --- [           main] c.f.letsencrypt.WildcardApplication      : No active profile set, falling back to default profiles: default
letsencrypt    | 2021-09-14 06:22:56.491  INFO 1 --- [           main] c.f.letsencrypt.WildcardApplication      : Started WildcardApplication in 2.635 seconds (JVM running for 3.773)
letsencrypt    | 2021-09-14 06:22:56.504  INFO 1 --- [           main] c.f.letsencrypt.WildcardApplication      : Starting up...
letsencrypt    | 2021-09-14 06:22:56.505  INFO 1 --- [           main] c.f.letsencrypt.WildcardApplication      : domain=futlabs.com
letsencrypt    | 2021-09-14 06:22:56.505  INFO 1 --- [           main] c.f.letsencrypt.WildcardApplication      : dnsProvider=alibaba
letsencrypt    | 2021-09-14 06:22:56.506  INFO 1 --- [           main] c.f.letsencrypt.WildcardApplication      : stage=TEST
letsencrypt    | 2021-09-14 06:23:00.513  INFO 1 --- [           main] c.f.letsencrypt.WildcardApplication      : Challenge for domain futlabs.com ...
letsencrypt    | 2021-09-14 06:23:01.510  INFO 1 --- [           main] c.f.letsencrypt.AlibabaDnsProvider       : Set TXT value to _acme-challenge.futlabs.com has been completed.
letsencrypt    | 2021-09-14 06:23:01.709  INFO 1 --- [           main] c.f.letsencrypt.WildcardApplication      : Your DNS settings are not yet in effect, wait an additional 30 seconds...

```

## 3. Get the certificate file

After you run the task sucessfully, you will get two files in the _certificate_ directory: 

- _domain-chain.crt_
The certificate issued by Let's Encrypt.

- _domain.key_
You domain private key.

## 4. A sample config in Nginx 

```

server {
    listen      80;
    server_name futlabs.com www.futlabs.com subdomain.futlabs.com;
    return       301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name futlabs.com www.futlabs.com subdomain.futlabs.com;

    client_max_body_size 5M;

    ssl_certificate /etc/nginx/certs/futlabs.com/domain-chain.crt;
    ssl_certificate_key /etc/nginx/certs/futlabs.com/domain.key;

    root /var/www/html;
}

```

