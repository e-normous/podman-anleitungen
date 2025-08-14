# Anleitung für die Erstellung eines podman Quadlets
(Alternative zu docker-compose und --restart=always)

vgl. ressourcen hier:
1. redhat.com/en/blog/quadlet-podman
2. giacomo.coletto.io/blog/podman-quadlets/
3. mag37.org/posts/guide_podman_quadlets/

## Vorbereitungsaufgaben
1. Erstellung eines Ordners, wo die quadlets gespeichert werden (unter dem entsprechenden User, root auch möglich)
    > mkdir -p $HOME/.config/containers/systemd
2. Lasse podman täglich nach aufdatierten images suchen
    > podman system migrate
    
    > systemctl --user enable --now podman-auto-update
3. Kreiere ein ".container" unit file, welches das container image laufen lässt (CONTAINERNAME ist dabei der Name des entsprechenden Containers, zu finden z.B. über "podman container ls -a")
    > nano $HOME/.config/containers/systemd/CONTAINERNAME.container
4. Gebe folgende Informationen an und speichere das file (ctrl + x und dann y)
    ```Dockerfile
    [Unit]
    Description=FREIER BESCHRIEB DES CONTAINERS/SERVICES

    [Container]
    Image=localhost/CONTAINER-IMAGE:latest
    ContainerName=CONTAINERNAME
    PublishPort=JE NACHDEM ABHÄNGIG (z.B. bei einer streamlit-app --> 8504:8504)

    [Service]
    Restart=always

    [Install]
    WantedBy=default.target

    ```
5. Reload den systemd daemon
    > systemctl --user daemon-reload
    
    Nun wird ein file, basierend auf CONTAINERNAME.container (z.B. tarifbewertung.container), gebildet mit dem namen CONTAINER.service (z.B. tarifbewertung.service)
6. Status anschauen (kann auch im Verlauf immer wieder gemacht werden)
    > systemctl --user status CONTAINERNAME.service
7. Den Service starten
    > systemctl --user start CONTAINERNAME.service
8. Nun muss noch der User (z.B. "pi") enabled werden, dass er beim reboot und nach logout noch services starten kann
    > loginctl show-user pi

    > sudo loginctl enable-linger pi

    > loginctl show-user pi

    Es sollte beim ersten Befehl bei "Linger=no" stehen und nach dem mittleren Befehl sollte "linger=yes" stehen.
Nun sollte die Applikation funktionieren und auch bei einem reboot neustarten.