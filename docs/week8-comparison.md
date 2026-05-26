# Week 8: On-Premise Docker vs Cloud Run Comparison

## Comparison Table

| Dimension | On-Premise Docker (Wks 3-5) | Cloud Run (Week 8) |
|---|---|---|
| Infrastructure setup | 3 VMs created, Docker installed on each | no vm is needed we just push a container image|
| Deployment command | SSH → docker build → docker run | there is no need to ssh gcloud builds submits plus terraform |
| TLS / HTTPS | Not configured | in week 8 https is automatically provided by google|
| Scaling approach | Manual — redeploy or add VMs | you can automatically scale up and down to zero|
| Port management | Ports 5000/5001/5002 per environment | port management is not needed it is handled automatically |
| Cost when idle | VM running 24/7 regardless of traffic | it will cost nothing when idle it just scales to zero |
| Rollback | Re-deploy previous image manually | you can deploy previous image through gcloud or terraform|
| Secrets management | GitHub Secrets → env vars in workflow | github secrets → env vars through OIDC workflow|

## Reflection Questions

**Q1:** i believe the step that required more step was docker because i had to ssh into each vm build the docker image on that machine restart the container manual. Cloud run gets rids of all that unnecessary steps now i just push the image and it deploys automatically.

**Q2:** with premise docker i have to ssh into the vm and check which container is running to find the version but with cloud run and commit SHA tagging every image had the exact git commit inits tag so i can look at the running image tag and match it to a specific commit in git log.

**Q3:** when cloud run scales to zero there will be no process running attackers have no process to attack on premise vm running 24/7 open ports and running services are exposed to potential attacks even when nobody is using the app

**Q4:** attack surface that was eliminated when OIDC workflow replaced the SSH key secrets from Weeks 3-5 was eliminating long lived secret completely because now there are no keys to steal or accidently expose instead OIDC uses short lived token that expire after each workflow run.

