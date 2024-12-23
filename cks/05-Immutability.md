## Mutable vs Immuttable Infrastructure

**Mutable** - Applying in-place upgrades, meaning log into container shell, and then upgrade version. 

**Immutable** - Delete the old pod, and recreate new pod with newer version using rolling-updates.

Containers are meant to be immutable in nature. 

## Ways to ensure immutability

1. Ensure we cannot write to the file-system using `readOnlyRootFilesystem` SecurityContext
```
spec:
  container:
    image: nginx
    securityContext:
      readOnlyRootFilesystem: true
```

This might not be suitable for all, as in many cases, applications would need to write to root FS to function properly.
For eg, `nginx` needs to be able to write to `/var/run` & `/var/cache/nginx`  [can be seens from error logs]. To overcome this, we can create `volumeMounts` to ensure those paths can be written into.

```
spec:
  container:
    image: nginx
    securityContext:
      readOnlyRootFilesystem: true
    volumes:
      - name: cache-vol
        mountPath: /var/cache/nginx
      - name: run-vol
        mountPath: /var/run
    volumeMounts:
      - name: cache-vol
        emptyDir: {}
      - name: run-vol
        emptyDir: {}
```

2. **DO NOT** use privileged container as it can allow one to make changes despite `readOnlyRootFilesystem`. Set `privileged: false` in the SecurityContext.
3. Always run as non-root user.

```
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
```
