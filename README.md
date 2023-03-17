# System Juliet Data Warehouse

This project represents Postgres Data warehouse as run on Kubernetes via Helm, as a part of the overall https://github.com/jvalentino/sys-juliet project. For system details, please see that location.

Prerequisites

- Git
- Helm
- Minikube
- psql
- Pgadmin

All of these you can get in one command using this installation automation (if you are on a Mac): https://github.com/jvalentino/setup-automation

## Stack

Postgres

> PostgreSQL, also known as Postgres, is a free and open-source relational database management system emphasizing extensibility and SQL compliance. It was originally named POSTGRES, referring to its origins as a successor to the Ingres database developed at the University of California, Berkeley

https://en.wikipedia.org/wiki/PostgreSQL

## Deployment

Prerequisites

- None

To re-install it, forward ports, and then verify it worked, use:

```bash
./deploy.sh
```

...which automatically runs the verification script to forward ports.

### deploy.sh

```bash
#!/bin/sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm delete --wait pg-secondary || true
helm install pg-secondary \
	--wait \
	--set auth.postgresPassword=postgres \
	--set auth.username=postgres \
	--set auth.database=dw \
	bitnami/postgresql
sh -x ./verify.sh
```

This works by using the Helm repository of `https://charts.bitnami.com/bitnami` for the configuration of `bitnami/postgresql` to run a single Postgres pod that is exposed on port 5432.

### verify.sh

```bash
#!/bin/sh
mkdir build || true
kubectl port-forward --namespace default svc/pg-secondary-postgresql 5433:5432 > build/pg-secondary-postgresql.log 2>&1 &
psql -d postgresql://postgres:postgres@localhost:5433/dw -c "select now()"

while [ $? -ne 0 ]; do
    kubectl port-forward --namespace default svc/pg-secondary-postgresql 5433:5432 > build/pg-secondary-postgresql.log 2>&1 &
    psql -d postgresql://postgres:postgres@localhost:5433/dw -c "select now()"
    sleep 5
done
```

## Runtime

### Kubernetes Dashboard

It will show on the Dashboard as a "Stateful Set" because this specific service involve storing data on the file system.

[![01](https://github.com/jvalentino/sys-golf/raw/main/wiki/07.png)](https://github.com/jvalentino/sys-golf/blob/main/wiki/07.png)

### pgadmin

You can then verify it is working by using pgadmin:

[![01](https://github.com/jvalentino/sys-golf/raw/main/wiki/08.png)](https://github.com/jvalentino/sys-golf/blob/main/wiki/08.png)

