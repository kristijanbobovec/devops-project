# Runbook — Troubleshooting guide

## Scenarij 1 — Pod ne pokreće se (CrashLoopBackOff)

**Simptom:**
`````````````bash
kubectl get pods -n ticketing
# NAME    READY   STATUS             RESTARTS
# api-xx  0/1     CrashLoopBackOff   5
\```

**Dijagnostika:**
````````````bash
kubectl logs deployment/api -n ticketing
kubectl describe pod -l app=api -n ticketing
\```

**Rješenje:**
```````````bash
kubectl rollout restart deployment/api -n ticketing
\```

## Scenarij 2 — Pad baze (PostgreSQL)

**Simptom:** API vraća 500, worker ne procesira narudžbe

**Dijagnostika:**
``````````bash
kubectl get pods -n ticketing -l app=postgres
kubectl logs deployment/postgres -n ticketing
\```

**Rješenje:**
`````````bash
kubectl rollout restart deployment/postgres -n ticketing
kubectl rollout status deployment/postgres -n ticketing
\```

**Validacija:**
````````bash
curl $(minikube service api -n ticketing --url)/healthz
\```

## Scenarij 3 — Loš image tag (ImagePullBackOff)

**Simptom:** Pod ostaje u ImagePullBackOff statusu

**Dijagnostika:**
```````bash
kubectl describe pod -l app=api -n ticketing
\```

**Rješenje:**
``````bash
kubectl rollout undo deployment/api -n ticketing
\```

## Scenarij 4 — Neispravan Secret

**Simptom:** Greška autentifikacije prema bazi

**Dijagnostika:**
`````bash
kubectl get secret ticketing-secret -n ticketing
kubectl get secret ticketing-secret -n ticketing -o jsonpath='{.data.POSTGRES_PASSWORD}' | base64 -d
\```

**Rješenje:**
````bash
kubectl delete secret ticketing-secret -n ticketing
kubectl apply -f k8s/secret.yaml
kubectl rollout restart deployment/api -n ticketing
\```

## Korisne naredbe

```bash
kubectl get all -n ticketing
kubectl logs deployment/api -n ticketing -f
kubectl exec -it deployment/api -n ticketing -- sh
kubectl get events -n ticketing --sort-by='.lastTimestamp'
kubectl top pods -n ticketing
\```
