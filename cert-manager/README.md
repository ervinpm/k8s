# Setting up Let's Encrypt using cert manager in Kubernetes Baremetal

After setting up your very own Kubernetes instance, one of the first things that you want to do is spin up a simple web server like nginx. You'll then notice that securing the server (https) will require you to issue certificates. However when you generate a self-signed one, the browser will still complain that it's not as good as the real thing. Certificate Authorities will cost you money and you don't want to spend a lot if you are just doing this to learn stuff. Let's Encrypt lets you do this for free by providing an Automated Certificate Management Environment (ACME). Below are my notes on how I did it.

## Installing cert manager
Here is the yaml to install cert manager:

```bash
$ kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.2.0/cert-manager.yaml
```

## Testing the installation
To test if it worked, use a self signed certificate

```bash
$ kubectl apply -f test-resources.yaml
```

After that, you can confirm by describing the created resources:

```bash
$ kubectl describe certificate -n cert-manager-test
```

You should have something like this:

```bash
Spec:
  Common Name:  me.com
  Issuer Ref:
    Name:       test-selfsigned
  Secret Name:  selfsigned-cert-tls
Status:
  Conditions:
    Last Transition Time:  2021-03-05T18:14:30Z
    Message:               Certificate is up to date and has not expired
    Reason:                Ready
    Status:                True
    Type:                  Ready
  Not After:               2021-04-09T18:14:21Z
Events:
  Type    Reason      Age   From          Message
  ----    ------      ----  ----          -------
  Normal  CertIssued  5s    cert-manager  Certificate issued successfully
```

After that, you may cleanup the test resources

```bash
$ kubectl delete -f test-resources.yaml
```

### Installing letsencrypt

This is a very easy step just execute the following

```bash
$ kubectl apply -f letsencrypt-cluster-issuer.yaml
```

### References
https://cert-manager.io/docs/installation/kubernetes/

