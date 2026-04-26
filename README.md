# Kolokwium

**Zadanie 1**
Wybór miasta został zaprogramowany w JS (Node). Plik `server.js` zawiera kod wraz z komenatrzami

___
**Zadanie 2**
W pliku Dockerfile znajduje sie kod wraz z komentarzami 
___
**Zadanie 3**
Polecenie na uruchomienie procesu budowania obrazu 
```
docker build -t aplikacja:v1.0.0 .
```
Polecenie na uruchomienie zbudowanego obrazu
```
docker run -d --name aplikacja -p 3000:3000 aplikacja:v1.0.0
```
Uzyskanie logów z informacjami 
```
docker logs aplikacja
Autor aplikacji: Martyna Debowczyk
Data uruchomienia: 4/26/2026
Aplikacja działa na porcie 3000
```
Liczba warstw: 1 (z tego wzgledu że zdecydowana większosć poszła z cacha)
```
 => [internal] load build definition from Dockerfile                                                               0.0s
 => => transferring dockerfile: 1.40kB                                                                             0.0s
 => [internal] load metadata for docker.io/library/node:alpine3.23                                                 1.2s
 => [auth] library/node:pull token for registry-1.docker.io                                                        0.0s
 => [internal] load .dockerignore                                                                                  0.0s
 => => transferring context: 2B                                                                                    0.0s
 => [internal] load build context                                                                                  0.0s
 => => transferring context: 120B                                                                                  0.0s
 => [prod 1/7] FROM docker.io/library/node:alpine3.23@sha256:bdf2cca6fe3dabd014ea60163eca3f0f7015fbd5c7ee1b0e9ccb  0.0s
 => => resolve docker.io/library/node:alpine3.23@sha256:bdf2cca6fe3dabd014ea60163eca3f0f7015fbd5c7ee1b0e9ccb4ced6  0.0s
 => CACHED [prod 2/7] RUN apk add --update --no-cache curl &&     rm -rf /etc/apk/cache                            0.0s
 => CACHED [prod 3/7] RUN mkdir -p /home/node/app                                                                  0.0s
 => CACHED [prod 4/7] WORKDIR /home/node/app                                                                       0.0s
 => CACHED [app_build 1/7] ADD alpine-minirootfs-3.23.3-aarch64.tar /                                              0.0s
 => CACHED [app_build 2/7] RUN apk update &&     apk upgrade &&     apk add --no-cache nodejs npm &&     rm -rf /  0.0s
 => CACHED [app_build 3/7] RUN addgroup -S node && adduser -S node -G node                                         0.0s
 => CACHED [app_build 4/7] WORKDIR /home/node/app                                                                  0.0s
 => CACHED [app_build 5/7] COPY --chown=node:node package.json ./package.json                                      0.0s
 => CACHED [app_build 6/7] RUN npm install                                                                         0.0s
 => CACHED [app_build 7/7] COPY --chown=node:node server.js ./server.js                                            0.0s
 => CACHED [prod 5/7] COPY --from=app_build --chown=node:node /home/node/app/package.json ./                       0.0s
 => CACHED [prod 6/7] COPY --from=app_build --chown=node:node /home/node/app/node_modules ./node_modules           0.0s
 => CACHED [prod 7/7] COPY --from=app_build --chown=node:node /home/node/app/server.js ./                          0.0s
 => exporting to image                                                                                             0.0s
 => => exporting layers                                                                                            0.0s
 => => exporting manifest sha256:198a0a5f2baa4248415629cc39a69873e116c8455821e6931f791e2f1feba1af                  0.0s
 => => exporting config sha256:791c352322d4c92a5cbcb680a86a9f77dfeee149ca514648bcd5bb1cdba6c4cb                    0.0s
 => => exporting attestation manifest sha256:5c7fc3ef83a76707ee21ad9055fb0d4e5e5b09144e14b4feb8475b0d2be3d359      0.0s
 => => exporting manifest list sha256:7e9690baaea5a8fe60b2a12979a1958eea301e09853a419b510fc4ed8b55d901             0.0s
 => => naming to docker.io/library/aplikacja:v1.0.0                                                                0.0s
 => => unpacking to docker.io/library/aplikacja:v1.0.0
```
___
**ZADANIA DODATKOWE**
Przeskanowanie Scoutem obrazu:
```
docker scout quickview aplikacja:v1.0.0
    i New version 1.20.4 available (installed version is 1.20.1) at https://github.com/docker/scout-cli
    ✓ Image stored for indexing
    ✓ Indexed 249 packages
    ✓ Provenance obtained from attestation

    i Base image was auto-detected. To get more accurate results, build images with max-mode provenance attestations.
      Review docs.docker.com ↗ for more information.

 Target             │  aplikacja:v1.0.0  │    0C     3H    11M     1L  
   digest           │  7e9690baaea5      │                             
 Base image         │  node:25-alpine    │    0C     1H     3M     0L  
 Updated base image │  node:24-alpine    │    0C     1H     3M     0L  
                    │                    │                             

What's next:
    View vulnerabilities → docker scout cves aplikacja:v1.0.0
    View base image update recommendations → docker scout recommendations aplikacja:v1.0.0
    Include policy results in your quickview by supplying an organization → docker scout quickview aplikacja:v1.0.0 --org <organization>
```
W aplikacji wykryto 3 wysokie podatnosci:
- `CVE-2026-3805`, podatność tyczy się curla. Wystepuje w momencie jesli wystąpia dwa requesty SMB- w nasyzm przypakdu nie korzystamyz  protokołu SMB stąd podatnośc nas nie dotyczy 
- `CVE-2026-33671`, luka dotyczy `picomatch`, w programie nie użyto tejże biblioteki
- `CVE-2026-27135` luka z HTTP, nie została wypuszczona naprawiona wersja

Polecenie `docker scout recommendations aplikacja:v1.0.0` zaproponowało aktualizacje obrazu `alpine` do wersji 23 i 22, natomiat te wersje również posiadają te same "wysokie" podatnosci

Stworzenie buildera:
```
docker buildx create --name builder --driver docker-container --bootstrap
[+] Building 55.1s (1/1) FINISHED                                                                                       
 => [internal] booting buildkit                                                                                   55.1s
 => => pulling image moby/buildkit:buildx-stable-1                                                                54.7s
 => => creating container buildx_buildkit_builder0                                                                 0.4s
builder
```
Budowanie:
```
docker buildx build --platform linux/amd64,linux/arm64 --tag sabnaria/aplikacja:v1.0.0 --push .
[+] Building 13.3s (7/33)                                                                      docker-container:builder
 => [internal] load build definition from Dockerfile                                                               0.0s
 => => transferring dockerfile: 1.40kB
```
Sprawdzenie manifestu:
```
  Name:        docker.io/sabnaria/aplikacja:v1.0.0@sha256:565dedce3069291ead002abe78079b4450cdf16d2904f06c35e5af9b83885d8d
  MediaType:   application/vnd.oci.image.manifest.v1+json
  Platform:    linux/amd64
               
  Name:        docker.io/sabnaria/aplikacja:v1.0.0@sha256:9b0eefa0dc05788cc8420e98d03ce88ec16d9358692b8e8ca799ca9aab713003
  MediaType:   application/vnd.oci.image.manifest.v1+json
  Platform:    linux/arm64
```
Manifest zawiera deklaracje dla dwóch platform sprzętowych 
