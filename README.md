# Platform GitOps (Argo CD) — proto-labnum

Ce dépôt contient les **manifests Argo CD** pour piloter l’infrastructure de votre cluster :
- **Cilium** (Helm) + **config L2 / LoadBalancer** (CRDs Cilium)
- **Longhorn** (Helm)
- **Ingress-NGINX** (Helm, IP LB 10.10.0.101)
- **Argo CD** (Helm, exposé via Ingress `argocd-labnum.aphp.fr`)
- **Gitea** (Helm, `gitea-labnum.aphp.fr`)
- **Harbor** (Helm, `harbor-labnum.aphp.fr`)
- **Jenkins** (Helm, `jenkins-labnum.aphp.fr`)

> Hypothèse : TLS est **terminé sur NGINX du Proxmox**. Dans le cluster, les Ingress restent en HTTP.

## Démarrage (une fois Argo CD installé)

1. Ajoutez ce repo dans Argo CD (Settings → Repositories).
2. Modifiez `apps/platform-root.yaml` et remplacez `REPO_URL` par l’URL Git de **ce** dépôt (ex: `http://gitea-labnum.aphp.fr/platform/platform.git`).
3. Appliquez le projet et la racine :
   ```bash
   kubectl apply -f ressources/platform/projects/platform-project.yaml
   kubectl apply -f ressources/platform/apps/platform-root.yaml
   ```
4. Dans Argo CD, synchronisez `platform-root`. Les Applications enfants (Cilium, Longhorn, Ingress-NGINX, ArgoCD, Gitea, Harbor, Jenkins) apparaîtront et se synchroniseront automatiquement.

> Si vous avez déjà installé certaines plateformes via Helm **manuellement**, vous pouvez soit :
> - les désinstaller puis laisser Argo CD les réinstaller, **ou**
> - conserver les mêmes **releaseName** & namespaces pour qu’Argo “reprenne” la main plus facilement.

## Notes importantes

- **Cilium L2/LB** : `cilium/config/cilium-l2-and-lb.yaml` configure le **pool 10.10.0.100-150** et l’annonce L2 sur **eth0** côté **Workers** (les masters sont exclus).
- **Ingress-NGINX** : IP du LoadBalancer **10.10.0.101** (annoncée L2 par Cilium). Le NGINX du Proxmox reverse-proxy vers cette IP.
- **Argo CD** : mode `--insecure` derrière l’Ingress (HTTP interne), le TLS est géré par NGINX du Proxmox en frontal.
- **Harbor** : `expose.tls.enabled=false` (TLS en frontal), `externalURL` en **https** pour cohérence côté navigateur.
