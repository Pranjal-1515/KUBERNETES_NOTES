# ConfigMaps and Secrets

* **ConfigMap:** A ConfigMap is used to store non-sensitive configuration data in key-value pairs. 
                 It allows you to decouple configuration from your application code, making it easier to manage and update configurations without rebuilding your application.

* **Secrets:** Secrets are similar to ConfigMaps but are specifically designed to store sensitive information, like passwords, tokens, or SSH keys. 
               Secrets are encoded and can be accessed securely by applications without exposing the actual sensitive data in plaintext.

## USING THE KUBECTL CREATE CONFIGMAP COMMAND
You can define the map’s entries by passing literals to the kubectl command or you
can create the ConfigMap from files stored on your disk. Use a simple literal first:
```bash
   $ kubectl create configmap fortune-config --from-literal=sleep-interval=25
     configmap "fortune-config" created
```
<br>

* **YAML file for ConfigMap**
  
![image](https://github.com/user-attachments/assets/d406cdba-6943-42f3-8a44-aa3e78ea827d)


