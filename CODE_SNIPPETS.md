## Code Snippets from the Book

All of the code snippets are listed below so you can easily cut-n-paste them.

### Chapter 6 - Open Technical Practices - Beginnings, Srarting Right

Page 159:
```bash
helm repo add redhat-cop \
  https://redhat-cop.github.io/helm-charts
```
Page 160:
```bash
crc start -c 4 -m 12288
```
Page 161:
```bash
oc login <cluster_api> -u <name> -p <password>
oc new-project example
helm install my-jenkins redhat-cop/jenkins
```
Page 162:
```bash
helm fetch redhat-cop/jenkins --version 1.0.2
helm template test jenkins-1.0.2.tgz
oc get secrets -n example | grep helm
oc get pods --watch -o wide -n example
```
Page 163:
```bash
helm show values redhat-cop/jenkins
```
Page 164:
```bash
oc get route jenkins
```
Page 164:
```bash
oc get route jenkins
```

### Chapter 7 - Open Technical Practices - The Midpoint

Page 184:
```bash
git clone https://github.com/petbattle/ubiquitous-journey.git
tree ubiquitous-journey
```
Page 187:
```bash
helm template bootstrap --dependency-update -f \
  bootstrap/values-bootstrap.yaml bootstrap
helm upgrade --install bootstrap-journey \
  -f bootstrap/values-bootstrap.yaml \
  bootstrap --create-namespace --namespace labs-bootstrap
```
Page 188:
```bash
oc get pods -n labs-ci-cd
oc get routes argocd-server -n labs-ci-cd
```
Page 189:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
  metadata:
  name: bootstrap-journey
namespace: labs-ci-cd
spec:
  destination:
    namespace: labs-bootstrap
    server: https://kubernetes.default.svc
  project: default
  source:
    helm:
      parameters:
      - name: argocd-operator.ignoreHelmHooks
        value: "true"
      valueFiles:
      - values-bootstrap.yaml
    path: bootstrap
    repoURL: https://github.com/[YOUR FORK]/ubiquitous-journey.git
    targetRevision: main
  syncPolicy:
    automated: {}
```
Page 190:
```bash
argocd login $(oc get route argocd-server --template='{{ .spec.host }}' \
  -n labs-ci-cd):443 \
  --sso --insecure
argocd app create bootstrap-journey \
  --dest-namespace labs-bootstrap \
  --dest-server https://kubernetes.default.svc \
  --repo https://github.com/[YOUR FORK]/ubiquitous-journey.git \
  --revision main \
  --sync-policy automated \
  --path "bootstrap" \
  --helm-set argocd-operator.ignoreHelmHooks=true \
  --values "values-bootstrap.yaml"
```
Page 193:
```bash
helm template -f argo-app-of-apps.yaml ubiquitous-journey/ \
  | oc -n labs-ci-cd apply -f-
```

### Chapter 14 - Build It

Page 538:
```bash
helm repo add petbattle \
  https://petbattle.github.io/helm-charts
```
Page 539:
```bash
helm search repo pet-battle
wget https://raw.githubusercontent.com/petbattle/pet-battle/master/chart/values.yaml/tmp/values.yaml
```
Page 540:
```bash
oc login -u <username> --server=<server api url>
helm upgrade --install pet-battle-api \
  petbattle/pet-battle-api --version=1.0.15 \
  --namespace petbattle --create-namespace
helm upgrade --install pet-battle \
  petbattle/pet-battle --version=1.0.6 \
  -f /tmp/values.yaml --namespace petbattle
helm upgrade --install pet-battle-tournament \
  petbattle/pet-battle-tournament --version=1.0.39 \
  --set pet-battle-infra.install_cert_util=true \
  --timeout=10m \
  --namespace petbattle
```
Page 542:
```bash
cat <<EOF | oc apply -f -
apiVersion: helm.openshift.io/v1beta1
kind: HelmChartRepository
metadata:
  name: petbattle-charts
spec:
  name: petbattle
  connectionConfig:
    url: https://petbattle.github.io/helm-charts
EOF
```
Page 562:
```bash
cd applications/deployment
helm upgrade --install pet-battle-suite-stage -f \
  argo-app-of-apps-stage.yaml \
  --namespace labs-ci-cd .
helm upgrade --install pet-battle-suite-test -f \
  argo-app-of-apps-test.yaml \
  --namespace labs-ci-cd .
```
Page 566:
```bash
oc apply -n labs-ci-cd -f petbattle-jenkinspb-secret.yml
```
Page 568:
```bash
cat <<EOF | oc apply -f-
apiVersion: v1
stringData:
  password: GITHUB_TOKEN
  username: GITHUB_USERNAME
kind: Secret
metadata:
  labels:
    credential.sync.jenkins.openshift.io: "true"
  name: git-auth
  namespace: labs-ci-cd
type: kubernetes.io/basic-auth
EOF
```
Page 569:
```bash
cat << EOF > /tmp/super-dooper.yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: super-dooper
  labels:
    credential.sync.jenkins.openshift.io: "true"
type: "kubernetes.io/basic-auth"
stringData:
  password: "myGitHubToken"
  username: "donal"
EOF
kubeseal < /tmp/super-dooper.yaml > /tmp/sealed-super-dooper.yaml \
  -n labs-ci-cd \
  --controller-namespace labs-ci-cd \
  --controller-name sealed-secrets \
  -o yaml
```
Page 570:
```bash
cat /tmp/sealed-super-dooper.yaml
```
Page 571:
```bash
cat /tmp/sealed-super-dooper.yaml | oc apply -f- -n labs-ci-cd
```
Page 592:
```bash
tkn pr list -n labs-ci-cd
```
Page 595:
```bash
oc -n labs-ci-cd process pet-battle-api | oc -n labs-ci-cd create -f-
oc -n labs-ci-cd process pet-battle | oc -n labs-ci-cd create -f-
oc -n labs-ci-cd process pet-battle-tournament | oc -n labs-ci-cd create -f-
oc -n labs-ci-cd get route webhook \
  -o custom-columns=ROUTE:.spec.host --no-headers
```

### Chapter 15 - Run It

Page 603:
```bash
argocd login $(oc get route argocd-server --template='{{ .spec.host }}' \
  -n labs-ci-cd):443 --sso --insecure
argocd repo add \
  https://github.com/rht-labs/refactored-adventure.git
argocd app create knative\
  --repo https://github.com/rht-labs/refactored-adventure.git \
  --path knative/base \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace openshift-serverless \
  --revision master \
  --sync-policy automated
```
Page 604:
```bash
helm upgrade --install pet-battle-nsff \
  petbattle/pet-battle-nsff \
  --version=0.0.2 \
  --namespace petbattle
```
Page 605:
```bash
oc get pods --namespace petbattle
kn service create tensorflowserving-pb-nsff --namespace petbattle \
  --image=docker.io/tensorflow/serving:latest \
  --cmd "tensorflow_model_server" \
  --arg "--model_config_file=s3://models/models.config" \
  --arg "--monitoring_config_file=s3://models/prometheus_config.config" \
  --arg "--rest_api_port=8501" \
  --env S3_LOCATION=minio-pet-battle-nsff:9000 \
  --env AWS_ACCESS_KEY_ID=minio \
  --env AWS_SECRET_ACCESS_KEY=minio123 \
  --env AWS_REGION=us-east-1 \
  --env S3_REGION=us-east-1 \
  --env S3_ENDPOINT=minio-pet-battle-nsff:9000 \
  --env S3_USE_HTTPS="0" \
  --env S3_VERIFY_SSL="0" \
  --env AWS_LOG_LEVEL="3" \
  --port 8501 \
  --autoscale-window "120s"
```
Page 606:
```bash
kn route list
```
Page 607:
```bash
curl <url from kn route list>/v1/models/test_model
```
Page 608:
```bash
oc get pods \
  -l serving.knative.dev/configuration=tensorflowserving-pet-battle-nsff \
  --namespace petbattle
```
Page 609:
```bash
wget https://raw.githubusercontent.com/petbattle/pet-battle-nsff/main/requests/tfserving/nsff-negative.json
wget https://raw.githubusercontent.com/petbattle/pet-battle-nsff/main/requests/tfserving/nsff-positive.json
HOST=$(kn service describe tensorflowserving-pet-battle-nsff -o url) \
  /v1/models/test_model:predict
curl -s -k -H 'Content-Type: application/json \
  -H 'cache-control: no-cache' \
  -H 'Accept: application/json' \
  -X POST --data-binary '@nsff-negative.json' $HOST
curl -s -k -H 'Content-Type: application/json' \
  -H 'cache-control: no-cache' \
  -H 'Accept: application/json' \
  -X POST --data-binary '@nsff-positive.json' $HOST
HOST=$(kn service describe tensorflowserving-pet-battle-nsff -o url)
helm upgrade --install pet-battle-api petbattle/pet-battle-api \
  --version=1.0.15 \
  --set nsff.enabled=true \
  --set nsff.apiHost=${HOST##http://} \
  --set nsff.apiPort=80 --namespace petbattle
```
Page 616:
```bash
ls src/test/java/com/petbattle/containers/
head src/test/java/com/petbattle/integration/ITPetBattleAPITest.java
```
Page 616:
```bash
git commit --allow-empty -m "🍌 kickoff jenkins 🦆" && git push
```
Page 633:
```bash
oc get pdb
```
Page 651:
```bash
oc login <cluster_api> -u <name> -p <password>
git clone https://github.com/petbattle/pet-battle-analytics.git \
  && cd pet-battle-analytics
helm install pba charts/matomo
```
Page 653:
```bash
git clone git@github.com:petbattle/pet-battle.git && cd pet-battle
helm install nodownvote --set image_version=no-down-vote \
  --set route=false chart --namespace petbattle
oc get pods
helm install prod --set image_version=latest chart \
  --set a_b_deploy.svc_name=no-down-vote-pet-battle --namespace petbattle
oc get pods
```
Page 654:
```bash
oc get routes
helm upgrade prod --set image_version=latest chart \
  --set a_b_deploy.svc_name=no-down-vote-pet-battle \
  --set a_b_deploy.weight=10 --namespace petbattle
```
Page 656:
```bash
helm upgrade --install pet-battle-api-blue \
  petbattle/pet-battle-api --version=1.0.15 \
  --namespace petbattle --create-namespace
helm upgrade --install pet-battle-api-green \
  petbattle/pet-battle-api --version=1.0.15 \
  --set image_version=green \
  --namespace petbattle
```
Page 657:
```bash
oc expose service pet-battle-api-blue --name=bluegreen \
  --namespace petbattle
oc patch route/bluegreen --namespace petbattle -p \
  '{"spec":{"to":{"name":"pet-battle-api-green"}}}'
oc patch route/bluegreen --namespace petbattle -p \
  '{"spec":{"to":{"name":"pet-battle-api-blue"}}}'
```

### Chapter 16 - Own It

Page 668:
```bash
oc get pod -n petbattle | grep tournament
oc exec YOUR_TOURNAMENT_PODNAME -- curl localhost:8080/metrics
```
Page 670:
```bash
oc describe svc my-pet-battle-tournament
```
Page 672:
```bash
oc get routes
```
Page 673:
```bash
oc get pods --show-labels=true
```
Page 674:
```bash
oc get svc
```
Page 676:
```bash
oc get all -l app.kubernetes.io/part-of=petbattleworld \
  --server-print=false
```
Page 676:
```bash
java -jar tournament-1.0.0-SNAPSHOT-runner.jar
```
Page 680:
```bash
oc get route thanos-querier -n openshift-monitoring
```
Page 686/687:
```bash
helm upgrade \
  --install pet-battle-tournament \
  --version=1.0.39 \
  --set pet-battle-infra.install_cert_util=true \
  --set istio.enabled=true \
  --timeout=10m \
  --namespace petbattle
  petbattle/pet-battle-tournament
oc get deployment pet-battle-tournament -o yaml \
  --namespace petbattle
oc get pods
```
Page 697:
```bash
oc get subscriptions
```
Page 700:
```bash
oc get svc keycloak -o yaml
oc get secret sso-x509-https-secret -o yaml
```
Page 701:
```bash
oc get secret sso-x509-https-secret -o json \
  | jq -r '.data."tls.crt"' | base64 --decode \
  |openssl x509 -text -noout
```

### Appendix A – OpenShift Sizing Requirements for Exercises

Page 755:
```bash
crc start
```
Page 756:
```bash
openshift-install cluster create
```
Page 757:
```bash
crc start -c 4 -m 10240
crc start -c 4 -m 12288
crc start -c 4 -m 16384 -d 50
crc config set enable-cluster-monitoring true
```
Page 758:
```bash
crc start -c 4 -m 16384 -d 50
CRC_MACHINE_IMAGE=${HOME}/.crc/machines/crc/crc.qcow2
crc stop
cp ${CRC_MACHINE_IMAGE} ${CRC_MACHINE_IMAGE}.ORIGINAL
virt-resize --expand /dev/vda4 \
  ${CRC_MACHINE_IMAGE}.ORIGINAL ${CRC_MACHINE_IMAGE}
rm -f ${CRC_MACHINE_IMAGE}.ORIGINAL
crc start -c 4 -m 16384 -d 50
```
Page 759:
```bash
oc get storageclass
```
