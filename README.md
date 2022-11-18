# Deploy the Grav CMS with Hashicorp Nomad

[Grav](https://getgrav.org/) is a Fast, Simple, and Flexible file-based Web-platform. Using Nomad to deploy the the Grav open source flat-file CMS.


### Complete Job File

```bash
job "grav" {
  datacenters = ["dc1"]
    group "grav" {
        
        network {
            mode = "host"
            port "http" {
                to = 80
            }
        }

        update {
            max_parallel = 1
            min_healthy_time  = "10s"
            healthy_deadline  = "5m"
            progress_deadline = "10m"
            auto_revert = true
            auto_promote = true
            canary = 1
        }

        restart {
            attempts = 10
            interval = "5m"
            delay    = "25s"
            mode     = "delay"
        }

        volume "grav_config" {
            type      = "host"
            read_only = false
            source    = "grav_config"
        }

        service {
            name = "grav-frontend"
            port = "http"
            tags = [
                "frontend"
            ]

            check {
                name     = "HTTP Health"
                path     = "/"
                type     = "http"
                protocol = "http"
                interval = "10s"
                timeout  = "2s"
            }
        }

        task "grav-frontend" {
            driver = "docker"

            volume_mount {
                volume      = "grav_config"
                destination = "/config"
                read_only   = false
            }

            config {
                image = "linuxserver/grav:latest"
                ports = ["http"]
                network_mode = "host"
                force_pull = false
            }

            env {
              PUID= "1000"
              PGID= "1000"
              TZ = "America/New_York"
            }

            resources {
                cpu    = 100
                memory = 300
            }
        }
    }
}
```