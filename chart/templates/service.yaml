kind: Service     
apiVersion: v1     
metadata:
  name: {{ .Release.Name }}-service #from RELEASE NAME
spec:
  selector:
    app: {{ .Release.Name }}
  type: ClusterIP
  ports:         
  - name: http   
    protocol: TCP
    port: {{ .Values.pod.port }} #from Values
    targetPort: {{ .Values.pod.target_port }}  #from values