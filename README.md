# Traefik Setup met Docker

Deze handleiding laat zien hoe je Traefik, een krachtige reverse proxy, kunt instellen met Docker. Met Traefik kun je gemakkelijk meerdere Docker-containerapplicaties beheren en beveiligen zonder dat je elk afzonderlijk hoeft te configureren.

## Stap 1: Docker Configuratie

Zorg ervoor dat Docker op je systeem is geïnstalleerd en correct werkt voordat je verder gaat.

## Stap 2: Traefik Configuratie

1. Maak een nieuw bestand genaamd `traefik.yml` aan en voeg de volgende configuratie toe:

    ```yaml
    # traefik.yml

    log:
      level: INFO

    api:
      dashboard: true

    providers:
      docker:
        endpoint: "unix:///var/run/docker.sock"
        exposedByDefault: false

    entryPoints:
      web:
        address: ":80"

      # Voeg hier eventueel extra entry points toe voor HTTPS

    certificatesResolvers:
      letsencrypt:
        acme:
          email: "jouw_email@example.com"
          storage: "/letsencrypt/acme.json"
          httpChallenge:
            entryPoint: "web"
    ```

2. Vervang `"jouw_email@example.com"` door je eigen e-mailadres.

3. Zorg ervoor dat je een directory hebt gemaakt voor het opslaan van de `acme.json`-bestand, zoals aangegeven in de configuratie.

## Stap 3: Docker Labels

Bij het draaien van Docker-containerapplicaties die je wilt blootstellen via Traefik, voeg je de juiste Docker-labels toe. Hier is een voorbeeld van hoe je dit doet:

    ```bash
    docker run -d \
      --name <container-naam> \
      -l "traefik.enable=true" \
      -l "traefik.http.routers.<router-naam>.rule=Host(`<gewenste_hostnaam>`)" \
      -l "traefik.http.services.<service-naam>.loadbalancer.server.port=<poort>" \
      <image>
    ```

Vervang `<container-naam>`, `<router-naam>`, `<gewenste_hostnaam>`, `<service-naam>` en `<poort>` door de relevante waarden voor je applicatie.

## Stap 4: Traefik Starten

Start Traefik met behulp van Docker:

```bash
docker run -d \
  -p 80:80 \
  -p 8080:8080 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /pad/naar/traefik.yml:/traefik.yml \
  -v /pad/naar/letsencrypt:/letsencrypt \
  --restart=always \
  --name traefik \
  traefik:v2.0 --configFile=/traefik.yml
