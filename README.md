# keycloak-custom-theme

A quick & practical example of using initContainer to bootstrap a theme for SSO pods running atop OpenShift or Kubernetes

# Usage

Customize this, or another theme, as you see fit -- the only change between this theme and the original Red Hat theme is a login background image: if the theme is configured correctly via initContainer, you'll see an image of clouds when accessing your SSO service.

Configure an initContainer call in your existing SSO deploymentConfig:

```
      initContainers:
        - command: [ 'sh', '-c', 'curl -L -k -o theme.zip https://github.com/liveaverage/keycloak-custom-theme/archive/master.zip && unzip theme.zip -d /opt/eap/themes/rh-sso && mv /opt/eap/themes/rh-sso/*/* /opt/eap/themes/rh-sso/']
          image: >-
            registry.access.redhat.com/redhat-sso-7/sso72-openshift:latest
          imagePullPolicy: IfNotPresent
          name: init-theme
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          volumeMounts:
            - mountPath: /opt/eap/themes/rh-sso
              name: vol-theme
```

Also ensure you have a corresponding *emptyDir* defined in the existing *volumes* list:

```
      volumes:
        - name: sso-x509-https-volume
          secret:
            defaultMode: 420
            secretName: sso-x509-https-secret
        - name: sso-x509-jgroups-volume
          secret:
            defaultMode: 420
            secretName: sso-x509-jgroups-secret
        - emptyDir: {}
          name: vol-theme
```

This directory will be populated with theme files by the initContainer, and mounted at the default location of `/opt/eap/themes/rh-sso` by your SSO container, which should eliminate the need for PVs and keep your theming a bit more dynamic (especially important if you're deploying multiple SSO pods requiring the same theme).
