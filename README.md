grafana-stack-podman
=========
[![License: MIT](https://img.shields.io/badge/License-MIT-brightgreen.svg)](https://opensource.org/licenses/MIT)
[![GitHub repo size in bytes](https://img.shields.io/github/repo-size/LyaSchumacher/grafana-stack-podman.svg?logoColor=brightgreen)](https://github.com/LyaSchumacher/grafana-stack-podman)
[![GitHub last commit](https://img.shields.io/github/last-commit/LyaSchumacher/grafana-stack-podman.svg?logoColor=brightgreen)](https://github.com/LyaSchumacher/grafana-stack-podman)

Fork from: https://github.com/CastawayEGR/grafana-stack-podman

Grafana Stack for Metrics, Logs, & Traces using Podman

## Installation

> **_NOTE:_**  Prequesites: git, firewalld(up and running) & Podman
>              Knowledge about [Rootless networking](https://www.redhat.com/sysadmin/container-networking-podman) is recommended. 

Clone this repo locally on your RHEL/CentOS/Rocky Linux/Alma Linux machine.

```sh
git clone https://github.com/lyaschumacher/grafana-stack-podman.git
```

Change to the cloned repo directory.

```sh
cd grafana-stack-podman
```

> **_IMPORTANT:_**  
> To be able to log out of the user you need to enable linger
> ```sh
> loginctl enable-linger $USER
> ```
> 
> Maybe you need to adjust the ownership recursive of the 'grafana' directory

To use traefik on Port 80 and 443 make them available for user to use

In order to run Traefik without root, we need to give the ability for unprivileged users to bind privileged ports by modifying a kernel parameter. Use `sysctl` to do this (as root):

```sh
sysctl net.ipv4.ip_unprivileged_port_start=80
```

And to make it persistent, run (also as root):

```sh
echo "net.ipv4.ip_unprivileged_port_start=80" > /etc/sysctl.d/user_priv_ports.conf

After editing we can spin up traefik using Podman.

```sh
podman play kube traefik.yaml
```

Now we want to spin up the grafana stack.

```sh
podman play kube grafana-stack.yaml
```

> **_NOTE:_**  
> To destroy everything you can run following command
> ```
> podman play kube --down grafana-stack.yaml
> ```

We can verify the stack is up and running by running the following.

```sh
podman ps
```

Next up we can expose Grafana, Mimir, Loki, and Tempo via firewall-cmd.

```sh
sudo firewall-cmd --permanent --add-port={3000/tcp,3100/tcp,4317/tcp,4318/tcp,9009/tcp,9095/tcp,9096/tcp,9097/tcp,9411/tcp,14268/tcp}
```

```sh
sudo firewall-cmd --reload
```

Once deployed you can browse to the Grafana deployment by visiting ```http://{machine_ip}:3000``` and logging in with the default credentials.

## Grafana Configuration

Finally we want to configure our 3 newly deployed backends with the following information.

Prometheus: 

```http://{machine_ip}:9009/prometheus```

Loki: 

```http://{machine_ip}:3100```

Tempo: 

```http://{machine_ip}:3200```

That's it you are ready to start ingesting metrics, logs, and traces!

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License
[MIT](https://choosealicense.com/licenses/mit/)

