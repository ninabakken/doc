# Google Cloud Platform

| cluster    | environment | comment                                |
|:-----------|:------------|:---------------------------------------|
| `dev-gcp`  | development | selected ingresses publicly accessible |
| `prod-gcp` | production  | publicly accessible                    |
| `labs-gcp` | development | publicly accessible                    |

In GCP, we do not operate with a zone model like with the on-premise clusters. Instead, we rely on a [zero trust](../appendix/zero-trust.md) model with a service mesh. The only thing we differentiate on a cluster level is development and production.

The applications running in GCP need [access policy rules defined](../nais-application/access-policy.md) for every other service they receive requests from or sends requests to.

To access the GCP clusters, see [Access](../basics/access.md#google-cloud-platform-gcp).

## Access to GCP
In order to use GCP, a team is required to add their team in a PR to [navikt/teams](https://github.com/navikt/teams).
This will generate a namespace for the team in each cluster, and dev and prod GCP projects will be created.
The team's group is initially granted a restricted set of permissions in these projects, but have the ability to grant further permissions on demand using the [GCP console](https://console.cloud.google.com)
!!! warning
    With the ability to grant permissions, the team has full control of the team's GCP projects, and should take care when granting further permissions or enabling features and APIs.

## Accessing the application

Access is controlled in part by ingresses, which define where your application will be exposed as a HTTP endpoint. You can control where your application is reachable from by selecting the appropriate ingress domain.

!!! warning
    Make sure you understand where you expose your application, taking into account the state of your application, what kind of data it exposes and how it is secured. If in doubt, ask in \#nais or someone on the NAIS team.


You can control from where you application is reachable by selecting the appropriate ingress domain. If no ingress is selected, the application will not be reachable from outside the cluster.

### dev-gcp ingresses

| domain             | accessible from                   | description                                                                                                                          |
|:-------------------|:----------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------|
| ekstern.dev.nav.no | internet                          | manually configured, see [instructions below](#eksterndevnavno). URLs containing `/metrics`, `/actuator` or `/internal` are blocked. |
| dev.nav.no         | [naisdevice](../device/README.md) | development ingress for nav.no applications                                                                                          |
| dev.intern.nav.no  | [naisdevice](../device/README.md) | development ingress for non-public/internet-facing applications                                                                      |


#### ekstern.dev.nav.no

In order to allow an ingress `<app>.ekstern.dev.nav.no` to resolve, it must be configured in our loadbalancer config found on [GitHub](https://github.com/nais/gcp/blob/master/infrastructure/dev.tfvars).

You might see a `404 Page not found` error when visiting the [GitHub repository](https://github.com/nais/gcp/blob/master/infrastructure/dev.tfvars) for the load balancer config.
To gain access, do the following:

1. Go to [myapps.microsoft.com](https://account.activedirectory.windowsazure.com/r#/addApplications).
2. Find the `GitHub.com (nais)` app and add it to your account.

Add a new entry to `external_domains` under `rules` for `"gw-ekstern-dev-nav-no"`:

```terraform
external_domains = {
  ...
  "gw-ekstern-dev-nav-no" = {
    rules = [
      ...
      {
        description = "allow access to <app>.ekstern.dev.nav.no"
        priority    = "<previous value, incremented by 1>"
        action      = "allow"
        expr        = "request.headers['Host'] == '<app>.ekstern.dev.nav.no'"
        preview     = false
      },
    ]
  }
}
```

Commit the changes and create a pull request.

### prod-gcp ingresses

| domain           | accessible from                   | description                                                                                                          |
|:-----------------|:----------------------------------|:---------------------------------------------------------------------------------------------------------------------|
| nav.no           | internet                          | manually configured, contact at \#tech-sikkerhet. URLs containing `/metrics`, `/actuator` or `/internal` are blocked |
| intern.nav.no    | [naisdevice](../device/README.md) | used by non-public/internet-facing applications \(previously called adeo.no\).                                       |

More info about how DNS is configured for these domains can be found [here](../appendix/ingress-dns.md)

### labs-gcp ingresses

| domain       | accessible from | description              |
|:-------------|:----------------|:-------------------------|
| labs.nais.io | internet        | automatically configured |

!!! warning
    Note that the `labs-gcp` cluster is a separate cluster, primarily meant for deploying simple demos with mocks. 
    
    It does **not** provide any features or integrations that are present in the normal clusters (e.g. Kafka, Azure, etc.)

## ROS and PVK

When establishing an application on GCP, it is a great time to update its [Risikovurdering (*ROS*)](https://navno.sharepoint.com/sites/intranett-it/SitePages/Risikovurderinger.aspx) analysis. It is required to update the application's entry in the [*Behandlingsoversikt*](https://navno.sharepoint.com/sites/intranett-personvern/SitePages/Behandlingskatalog.aspx) when changing platforms. If both of these words are unfamiliar to your team, it's time to sit down and take a look at both of them.

Every application needs to have a *ROS* analysis. 
Applications handling personal information needs a [Personvernkonsekvens (*PVK*)](https://navno.sharepoint.com/sites/intranett-personvern/SitePages/PVK.aspx) analysis and an entry in the *Behandlingsoversikt*.

See also additional information about [*ROS*](../legal/app-ros.md) and [*PVK*](../legal/app-pvk.md) under Laws and regulations.

Questions about ROS can be directed to Leif Tore Løvmo or Line Langlo Spongsveen or posted in [#tryggnok](https://nav-it.slack.com/archives/CQ0D5HLSW). Questions about *Behandling* should be directed to [#behandlinskatalogen](https://nav-it.slack.com/archives/CR1B19E6L).

