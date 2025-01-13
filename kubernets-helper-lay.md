Sample of _helper.tpl
 
{{/* Logic to populate string based content */}}
 
{{/* Applicaion name used for image name, add suffix '-dev' when deploying dev environment */}}

{{- define "microservice.name" -}}

{{- .Values.name }}

{{- end -}}
 
{{/* Create chart name and version as used by chart label.*/}}

{{- define "microservice.chart" -}}

{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}

{{- end }}
 
{{/* Pre-define service */}}

{{- define "microservice.service" -}}

{{- if eq (.Values.deploy.service.overrideName) "" }}

{{- printf "%s-svc" (include "microservice.name" .) }}

{{- else }}

{{- .Values.deploy.service.overrideName }}

{{- end }}

{{- end }}
 
{{/* Pre-define ingress */}}

{{- define "microservice.ingress" -}}

{{- if eq (.Values.deploy.ingress.overrideName) "" }}

{{- printf "%s-ingress" (include "microservice.name" .) }}

{{- else }}

{{- .Values.deploy.ingress.overrideName }}

{{- end }}

{{- end }}
 
{{/* Service selector labels */}}

{{- define "microservice.selectorLabels" -}}

deployment: {{ include "microservice.name" . }}

app: {{ include "microservice.name" . }}-pod

clusterName: {{ include "microservice.name" . }}

tier: app

{{- end }}
 
{{/* Pod custom labels */}}

{{- define "microservice.podLabels" -}}

{{- range .Values.global.deploy.podLabels }}

{{ .key }}: {{ .value }}{{- end }}

{{- range .Values.deploy.container.podLabels }}

{{ .key }}: {{ .value }}{{- end }}

{{- end }}
 
{{/* Common annotations */}}

{{- define "microservice.annotations" -}}

meta.helm.sh/release-name: {{ .Release.Name }}

meta.helm.sh/release-namespace: {{ .Release.Namespace }}

{{- end }}
 
{{/* Common labels */}}

{{- define "microservice.labels" -}}

clusterName: {{ include "microservice.name" . }}

helm.sh/chart: {{ include "microservice.chart" . }}

meta.helm.sh/release-name: {{ include "microservice.name" . }}

meta.helm.sh/release-namespace: {{ .Release.Namespace }}

{{- if .Chart.AppVersion }}

app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}

{{- end }}

app.kubernetes.io/managed-by: {{ .Release.Service }}

{{- range .Values.global.deploy.resourceLabels }}

{{ .key }}: {{ .value }}

{{- end }}

{{- end -}}

 
