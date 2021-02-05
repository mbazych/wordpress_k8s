# Wordpress

Moim projektem jest system Wordpress postawiony na klastrze Kubernetesa w GKE - Google Kubernetes Engine.


## Dostępność

System dostępny jest pod tym [adresem](https://wordpress.mbazych.pl/)
Nie ma tam nic specjalnego ciekawego, jest to goły wordpress.

## Infrastruktura

### Klaster
Klaster k8s składa się z 3 instancji Compute Engine VM Instances w Google Cloud na którym chodzą usługi opisane w pliku *wordpress_cloudsql.yaml*.

### Kontenery
Projekt korzysta w dwóch kontenerów - sam wordpress oraz cloudsql proxy.


#### Wordpress
Do kontenera **Wordpress** użyłem gotowego obrazu [wordpress](https://hub.docker.com/_/wordpress). Na kontenerze działają usługi php oraz apache.
Dodatkowo do kontenera podpięta jest configmapa dodająca parametry do PHP tak aby sam Wordpress lepiej działał(zdefiniowana w pliku wordpress-config.yaml):
```
    file_uploads = On
    upload_max_filesize = 256M
    post_max_size = 256M
    memory_limit = 64M
    max_execution_time = 600
```
Jest również mount wordpress-persistent-storage(200G) podpięty pod /var/www/html z plikami Wordpressa.

#### cloudsql-proxy
**Cloudsql-proxy** korzysta z obrazu dostępnego w ramach Google Container Registry. (gcr.io/cloudsql-docker/gce-proxy:1.11) 
Jest to usługa zezwalająca na połączenie z bazą danych, tak naprawdę jedyne do czego służy kontener to do uruchomienia komendy:
```
cloud_sql_proxy
      -instances=$PROJECT_ID:europe-north1:$DB_INSTANCE_NAME=tcp:3306
      -credential_file=/secrets/cloudsql/key.json
```
### Sekrety
Klaster korzysta z następujących sekretów:
- **cloudsql-db-credentials** - user oraz hasło do bazy danych
- **cloudsql-instance-credentials** - dane bazy aby cloud_sql_proxy mogło się z nią połączyć
- **wordpress-crt** - certyfikat SSL dla Ingress

### Usługi
W ramach klastra uruchomiona jest jedna usługa typu **LoadBalancer** opisana w pliku *wordpress_service.yaml* która przyjmuje ruch na porcie 80 i kieruje ruch do podów. Dodatkowo mamy **Ingress** opisany w pliku *wordpress-ingress.yaml* który przyjmuje ruch z zewnątrz i kieruje do **LoadBalancera**. Przed Ingress jest również **Cloudflare**, który zapewnia przekierowania http -> https, certyfikat SSL oraz pozwala na ustawienie domeny dla naszej witryny. Rekord A domeny skierowany jest na publiczny IP **Ingress**..

### Baza danych
Baza danych to **MySQL 5.7** działający w ramach usługi **Cloud SQL** na Google Cloud Platform, jak wcześniej wspomniano, połączenie klastra jest możliwe dzięki **cloud_sql_proxy**.


## Autor
[Michał Bazych](https://www.linkedin.com/in/mbazych/)
