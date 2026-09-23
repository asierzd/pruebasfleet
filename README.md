# fleet-content — Contenido del repositorio git de Fleet

Este directorio contiene los manifiestos que Fleet desplegará en los clusters
edge (nodos K3s de Elemental). Copia su contenido a tu repositorio git externo
y haz push.

## Estructura

```
edge/
├── fleet.yaml        # Configuración del bundle Fleet (namespace por defecto)
├── namespace.yaml    # Crea el namespace fleet-demo en cada cluster
├── configmap.yaml    # Configuración versionada — editar para probar reconciliación
└── deployment.yaml   # Nginx que sirve una página con los valores del ConfigMap
```

## Configuración inicial

```bash
# 1. Copia al repo git
cp -r fleet-content/* /ruta/a/tu/repo-fleet/

# 2. Haz push
cd /ruta/a/tu/repo-fleet/
git add .
git commit -m "Fleet demo: configuración inicial"
git push
```

## Probar reconciliación (pausa/reanuda)

Una vez Fleet esté sincronizando:

```bash
# Ejecuta el script de prueba
./test-reconcile.sh

# El script pausará Fleet y te pedirá que hagas un cambio en git.
# Edita edge/configmap.yaml — cambia version y/o message.
# Edita también la anotación fleet-demo/config-version en deployment.yaml
# (mismo valor que version) para que K8s reinicie el pod con los nuevos valores.
```

Ejemplo de cambio en `edge/configmap.yaml`:
```yaml
data:
  version: "2.0"
  message: "Actualizado con Fleet pausado — ahora reconcilia"
```

Y en `edge/deployment.yaml`, actualiza las dos anotaciones `fleet-demo/config-version: "2.0"`.

## Verificar el resultado

En el cluster edge:
```bash
# Estado del ConfigMap
kubectl get configmap fleet-demo-config -n fleet-demo -o yaml

# Página nginx (port-forward)
kubectl port-forward svc/fleet-demo 8080:80 -n fleet-demo
curl http://localhost:8080
```
