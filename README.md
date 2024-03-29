# Istio-Peer-Authentication


**PeerAuthentication allows mTLS traffic for all workloads under namespace.  It defines how traffic will be tunneled (or not) to the sidecar.**

<br>

### Summary

In this lab, I will test Istio Injection + PeerAuthentication using multiple Scenatios.

1) Test Scenario 1 - Istio Injected app  <=> Non-Istio  <br>
2) Test Scenario 2 - Istio Injected app  <=> Istio Injected app  <br>
3) Test Scenario 3 - Istio Injected + PeerAuth app <=> Non-Istio   <br>
4) Test Scenario 4 - Istio Injected + PeerAuth app  <=> Istio Injected  <br>
5) Test Scenario 5 - PeerAuth to (Istio-System ns), Istio Injected <=> Non-Istio   <br>
6) Test Scenario 6 - PeerAuth to (Istio-System ns), Istio Injected <=> Istio Injected  <br>
7) Test Scenario 7 - PeerAuth app  <=> PeerAuth  <br>
8) Test Scenario 8 - Istio Injected + PeerAuth app <=> PeerAuth app  <br>
9) Test Scenario 9 - Istio Injected + PeerAuth app <=> Istio Injected + PeerAuth app  <br>
10) Test Scenario 10 - Istio Injected app  <=> PeerAuth  <br>
11) Should we use enable PeerAuth in "Istio-System"?

 <br>
 
### 1) Test Scenario 1

- Create istio-in-action namespace.
- Enable injection.
- deploy the sample application in Istio-in-action.

- Create test1 namespace.
- Deploy the sample application in test1.


![image](https://github.com/myathway-lab/Istio-Peer-Authentication/assets/157335804/11cbfb04-1907-48d6-8f40-c77871646485)

 <br>
 
- Let’s verify if the app from both namespaces can access each other.
  
> istion-in-action ⇒ test1
> 

```yaml
vagrant@istio-cluster:~$ kubectl exec -it  sleep-66b45d6984-xr6sc -n istio-in-action -- curl http://web-api.test1:8080
{
  "name": "web-api",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "10.244.1.5"
  ],
  "start_time": "2024-02-26T16:48:36.658868",
  "end_time": "2024-02-26T16:48:36.670555",
  "duration": "11.686418ms",
  "body": "Hello From Web API",
  "upstream_calls": [
    {
      "name": "recommendation",
      "uri": "http://recommendation:8080",
      "type": "HTTP",
      "ip_addresses": [
        "10.244.3.7"
      ],
      "start_time": "2024-02-26T16:48:36.661079",
      "end_time": "2024-02-26T16:48:36.669432",
      "duration": "8.353116ms",
      "body": "Hello From Recommendations!",
      "upstream_calls": [
        {
          "name": "purchase-history-v1",
          "uri": "http://purchase-history:8080",
          "type": "HTTP",
          "ip_addresses": [
            "10.244.2.7"
          ],
          "start_time": "2024-02-26T16:48:36.668955",
          "end_time": "2024-02-26T16:48:36.669074",
          "duration": "118.457µs",
          "body": "Hello From Purchase History (v1)!",
          "code": 200
        }
      ],
      "code": 200
    }
  ],
  "code": 200
}
vagrant@istio-cluster:~$ 
```

> test1  ⇒ istion-in-action
> 

```yaml
vagrant@istio-cluster:~$ kubectl exec -it  sleep-6ffdd98f9f-lvhn5 -n test1 -- curl http://web-api.istio-in-action:8080
{
  "name": "web-api",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "10.244.2.6"
  ],
  "start_time": "2024-02-26T16:53:35.975389",
  "end_time": "2024-02-26T16:53:36.046082",
  "duration": "70.693085ms",
  "body": "Hello From Web API",
  "upstream_calls": [
    {
      "name": "recommendation",
      "uri": "http://recommendation:8080",
      "type": "HTTP",
      "ip_addresses": [
        "10.244.1.4"
      ],
      "start_time": "2024-02-26T16:53:36.011749",
      "end_time": "2024-02-26T16:53:36.041113",
      "duration": "29.363938ms",
      "body": "Hello From Recommendations!",
      "upstream_calls": [
        {
          "name": "purchase-history-v1",
          "uri": "http://purchase-history:8080",
          "type": "HTTP",
          "ip_addresses": [
            "10.244.3.5"
          ],
          "start_time": "2024-02-26T16:53:36.026338",
          "end_time": "2024-02-26T16:53:36.026492",
          "duration": "154.194µs",
          "body": "Hello From Purchase History (v1)!",
          "code": 200
        }
      ],
      "code": 200
    }
  ],
  "code": 200
}
```

![image](https://github.com/myathway-lab/Istio-Peer-Authentication/assets/157335804/c51ca5af-0691-47c6-af3d-df8306196c47)

<br>

### 2) Test Scenario 2

- Create two namespaces
- Inject the Istio.
- deploy the applications.

![image](https://github.com/myathway-lab/Istio-Peer-Authentication/assets/157335804/4dbc8413-848c-4c0d-8500-d2032a61c0f5)


- Let’s verify if the applications are able to access.
  
> istio-in-action ⇒ test2
> 

```yaml
vagrant@istio-cluster:~$ kubectl exec -it sleep-66b45d6984-xr6sc -n istio-in-action -- curl http://web-api.test2:8080
{
  "name": "web-api",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "10.244.3.10"
  ],
  "start_time": "2024-02-26T17:04:22.171333",
  "end_time": "2024-02-26T17:04:22.213432",
  "duration": "42.09877ms",
  "body": "Hello From Web API",
  "upstream_calls": [
    {
      "name": "recommendation",
      "uri": "http://recommendation:8080",
      "type": "HTTP",
      "ip_addresses": [
        "10.244.2.9"
      ],
      "start_time": "2024-02-26T17:04:22.186861",
      "end_time": "2024-02-26T17:04:22.206239",
      "duration": "19.377575ms",
      "body": "Hello From Recommendations!",
      "upstream_calls": [
        {
          "name": "purchase-history-v1",
          "uri": "http://purchase-history:8080",
          "type": "HTTP",
          "ip_addresses": [
            "10.244.1.8"
          ],
          "start_time": "2024-02-26T17:04:22.199196",
          "end_time": "2024-02-26T17:04:22.200293",
          "duration": "1.096428ms",
          "body": "Hello From Purchase History (v1)!",
          "code": 200
        }
      ],
      "code": 200
    }
  ],
  "code": 200
}
```

> test2 ⇒ istio-in-action
> 

```yaml
vagrant@istio-cluster:~$ kubectl exec -it sleep-76878f8d59-5qk79 -n test2 -- curl http://web-api.istio-in-action:8080
{
  "name": "web-api",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "10.244.2.6"
  ],
  "start_time": "2024-02-26T17:05:47.187207",
  "end_time": "2024-02-26T17:05:47.207770",
  "duration": "20.563367ms",
  "body": "Hello From Web API",
  "upstream_calls": [
    {
      "name": "recommendation",
      "uri": "http://recommendation:8080",
      "type": "HTTP",
      "ip_addresses": [
        "10.244.1.4"
      ],
      "start_time": "2024-02-26T17:05:47.193032",
      "end_time": "2024-02-26T17:05:47.203807",
      "duration": "10.774252ms",
      "body": "Hello From Recommendations!",
      "upstream_calls": [
        {
          "name": "purchase-history-v1",
          "uri": "http://purchase-history:8080",
          "type": "HTTP",
          "ip_addresses": [
            "10.244.3.5"
          ],
          "start_time": "2024-02-26T17:05:47.197608",
          "end_time": "2024-02-26T17:05:47.200058",
          "duration": "2.449888ms",
          "body": "Hello From Purchase History (v1)!",
          "code": 200
        }
      ],
      "code": 200
    }
  ],
  "code": 200
}
```

![image](https://github.com/myathway-lab/Istio-Peer-Authentication/assets/157335804/035bae09-6b91-44a8-860a-8a8aa8b89b4f)


<aside>
💡 Even though the Istio Injection is enabled, since we haven’t driven Istio to do Istio provided functions, the pods are still able to access.

</aside>

<br>

### Test Scenario 3

**We will use below Policy to allow mTLS traffic for all workloads under namespace istio-in-action.**

- peerauth.yaml
    
    ```yaml
    apiVersion: security.istio.io/v1beta1
    kind: PeerAuthentication
    metadata:
      name: default
    spec:
      mtls:
        mode: STRICT
    ```

```yaml
vagrant@istio-cluster:~$ kubectl get po -n istio-in-action
NAME                                   READY   STATUS    RESTARTS   AGE
purchase-history-v1-75559f9c4f-hpfmf   2/2     Running   0          76m
recommendation-dbd4c764d-npjtm         2/2     Running   0          76m
sleep-66b45d6984-xr6sc                 2/2     Running   0          76m
web-api-5d758fb95-9bfn8                2/2     Running   0          76m
vagrant@istio-cluster:~$  kubectl get po -n test3
NAME                                  READY   STATUS    RESTARTS   AGE
purchase-history-v1-fc9446597-tthb9   1/1     Running   0          33s
recommendation-bb5cbdc9d-bpzzl        1/1     Running   0          61s
web-api-7c9755c6d-p7fcl               1/1     Running   0          54s
vagrant@istio-cluster:~$ kubectl get peerauthentication -n istio-in-action
NAME      MODE     AGE
default   STRICT   8m50s
vagrant@istio-cluster:~$ kubectl get peerauthentication -n test3
No resources found in test3 namespace.
```



> istio-in-action ⇒ test3
>
```yaml
vagrant@istio-cluster:~/kind-demo$ kubectl exec -it sleep-66b45d6984-xr6sc -n istio-in-action -- curl http://web-api.test3:8080
{
  "name": "web-api",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "10.244.2.10"
  ],
  "start_time": "2024-02-26T17:43:58.691200",
  "end_time": "2024-02-26T17:43:58.742815",
  "duration": "51.615176ms",
  "body": "Hello From Web API",
  "upstream_calls": [
    {
      "name": "recommendation",
      "uri": "http://recommendation:8080",
      "type": "HTTP",
      "ip_addresses": [
        "10.244.1.9"
      ],
      "start_time": "2024-02-26T17:43:58.702197",
      "end_time": "2024-02-26T17:43:58.741843",
      "duration": "39.645253ms",
      "body": "Hello From Recommendations!",
      "upstream_calls": [
        {
          "name": "purchase-history-v1",
          "uri": "http://purchase-history:8080",
          "type": "HTTP",
          "ip_addresses": [
            "10.244.1.10"
          ],
          "start_time": "2024-02-26T17:43:58.723005",
          "end_time": "2024-02-26T17:43:58.740703",
          "duration": "17.698253ms",
          "body": "Hello From Purchase History (v1)!",
          "code": 200
        }
      ],
      "code": 200
    }
  ],
  "code": 200
}
```
> test3 ⇒ istio-in-action
> 

```yaml
vagrant@istio-cluster:~$ kubectl exec -it sleep-6ffdd98f9f-xqbbl -n test3 -- curl http://web-api.istio-in-action:8080
curl: (56) Recv failure: Connection reset by peer
command terminated with exit code 56
```
![image](https://github.com/myathway-lab/Istio-Peer-Authentication/assets/157335804/f3279f70-d2ac-431a-bb79-abe639f12fb2)

<br>

### 4) Test Scenario 4
We will inject istio in test3 namespace & see if able to access. 
```yaml
vagrant@istio-cluster:~$ kubectl label ns test3 istio-injection=enabled
vagrant@istio-cluster:~$ kubectl get po -n test3
NAME                                   READY   STATUS    RESTARTS   AGE
purchase-history-v1-7765657599-gtrn8   2/2     Running   0          4m58s
recommendation-65f4d79d66-gm4zz        2/2     Running   0          4m55s
sleep-7bbf5f8cc9-mcrpn                 2/2     Running   0          4m51s
web-api-b8b4b874-kslnb                 2/2     Running   0          4m53s
vagrant@istio-cluster:~$ kubectl get po -n istio-in-action
NAME                                   READY   STATUS    RESTARTS   AGE
purchase-history-v1-75559f9c4f-hpfmf   2/2     Running   0          100m
recommendation-dbd4c764d-npjtm         2/2     Running   0          100m
sleep-66b45d6984-xr6sc                 2/2     Running   0          100m
web-api-5d758fb95-9bfn8                2/2     Running   0          100m
vagrant@istio-cluster:~$ kubectl get peerauthentication -n istio-in-action
NAME      MODE     AGE
default   STRICT   32m
vagrant@istio-cluster:~/kind-demo/hellocloud-native-box/istio-cop/1-start-istio/sample-apps$ kubectl get peerauthentication -n  test3
No resources found in test3 namespace.
vagrant@istio-cluster:~/kind-demo/hellocloud-native-box/istio-cop/1-start-istio/sample-apps$
```

```yaml
vagrant@istio-cluster:~$ kubectl exec -it sleep-66b45d6984-xr6sc -n istio-in-action -- curl http://web-api.te
st3:8080
{
  "name": "web-api",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "10.244.2.14"
  ],
  "start_time": "2024-02-26T18:03:33.128461",
  "end_time": "2024-02-26T18:03:33.168000",
  "duration": "39.53886ms",
  "body": "Hello From Web API",
  "upstream_calls": [
    {
      "name": "recommendation",
      "uri": "http://recommendation:8080",
      "type": "HTTP",
      "ip_addresses": [
        "10.244.1.13"
      ],
      "start_time": "2024-02-26T18:03:33.141988",
      "end_time": "2024-02-26T18:03:33.160831",
      "duration": "18.843063ms",
      "body": "Hello From Recommendations!",
      "upstream_calls": [
        {
          "name": "purchase-history-v1",
          "uri": "http://purchase-history:8080",
          "type": "HTTP",
          "ip_addresses": [
            "10.244.1.12"
          ],
          "start_time": "2024-02-26T18:03:33.153984",
          "end_time": "2024-02-26T18:03:33.154195",
          "duration": "211.104µs",
          "body": "Hello From Purchase History (v1)!",
          "code": 200
        }
      ],
      "code": 200
    }
  ],
  "code": 200
}
```

```yaml
vagrant@istio-cluster:~$ kubectl exec -it sleep-7bbf5f8cc9-mcrpn -n test3 -- curl http://web-api.istio-in-act
ion:8080
{
  "name": "web-api",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "10.244.2.6"
  ],
  "start_time": "2024-02-26T18:04:28.411206",
  "end_time": "2024-02-26T18:04:28.427353",
  "duration": "16.146617ms",
  "body": "Hello From Web API",
  "upstream_calls": [
    {
      "name": "recommendation",
      "uri": "http://recommendation:8080",
      "type": "HTTP",
      "ip_addresses": [
        "10.244.1.4"
      ],
      "start_time": "2024-02-26T18:04:28.416914",
      "end_time": "2024-02-26T18:04:28.423636",
      "duration": "6.722088ms",
      "body": "Hello From Recommendations!",
      "upstream_calls": [
        {
          "name": "purchase-history-v1",
          "uri": "http://purchase-history:8080",
          "type": "HTTP",
          "ip_addresses": [
            "10.244.3.5"
          ],
          "start_time": "2024-02-26T18:04:28.421283",
          "end_time": "2024-02-26T18:04:28.421419",
          "duration": "136.346µs",
          "body": "Hello From Purchase History (v1)!",
          "code": 200
        }
      ],
      "code": 200
    }
  ],
  "code": 200
}
```
![image](https://github.com/myathway-lab/Istio-Peer-Authentication/assets/157335804/926929f0-ea5a-47f8-84cb-93ff896df1b3)


<br>


### 5) Test Scenario 5
**We will enable PeerAuthentication in Istio-System.**  <br>

**Once we enabled the PeerAuth in Istio-system, all the namespaces automatically get PeerAuth.**

```yaml
vagrant@istio-cluster:~/kind-demo$ kubectl apply -f peerauth.yaml -n istio-system
[peerauthentication.security.istio.io/default](http://peerauthentication.security.istio.io/default) created
vagrant@istio-cluster:~/kind-demo$ kubectl get peerauthentication -n istio-system
NAME      MODE     AGE
default   STRICT   5s
vagrant@istio-cluster:~/kind-demo$ kubectl get po  -n test4
NAME                                  READY   STATUS    RESTARTS   AGE
purchase-history-v1-fc9446597-kslss   1/1     Running   0          3m13s
recommendation-bb5cbdc9d-pnfnd        1/1     Running   0          3m13s
sleep-6ffdd98f9f-vmzgj                1/1     Running   0          3m12s
web-api-7c9755c6d-slwvg               1/1     Running   0          3m12s
agrant@istio-cluster:~/kind-demo$ kubectl get po  -n istio-in-action
NAME                                   READY   STATUS    RESTARTS   AGE
purchase-history-v1-75559f9c4f-hpfmf   2/2     Running   0          118m
recommendation-dbd4c764d-npjtm         2/2     Running   0          117m
sleep-66b45d6984-xr6sc                 2/2     Running   0          117m
web-api-5d758fb95-9bfn8                2/2     Running   0          117m
```

```yaml
vagrant@istio-cluster:~/kind-demo$ kubectl exec -it sleep-66b45d6984-xr6sc -n istio-in-action -- curl http://web-api.test4:8080
{
  "name": "web-api",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "10.244.2.16"
  ],
  "start_time": "2024-02-26T18:23:04.782816",
  "end_time": "2024-02-26T18:23:04.799626",
  "duration": "16.810079ms",
  "body": "Hello From Web API",
  "upstream_calls": [
    {
      "name": "recommendation",
      "uri": "http://recommendation:8080",
      "type": "HTTP",
      "ip_addresses": [
        "10.244.1.14"
      ],
      "start_time": "2024-02-26T18:23:04.788226",
      "end_time": "2024-02-26T18:23:04.793583",
      "duration": "5.357656ms",
      "body": "Hello From Recommendations!",
      "upstream_calls": [
        {
          "name": "purchase-history-v1",
          "uri": "http://purchase-history:8080",
          "type": "HTTP",
          "ip_addresses": [
            "10.244.2.15"
          ],
          "start_time": "2024-02-26T18:23:04.792539",
          "end_time": "2024-02-26T18:23:04.792665",
          "duration": "126.558µs",
          "body": "Hello From Purchase History (v1)!",
          "code": 200
        }
      ],
      "code": 200
    }
  ],
  "code": 200
}
```

```yaml
vagrant@istio-cluster:~/kind-demo$ kubectl exec -it sleep-6ffdd98f9f-vmzgj  -n test4 -- curl http://web-api.istio-in-action:8080
curl: (56) Recv failure: Connection reset by peer
command terminated with exit code 56
```

![image](https://github.com/myathway-lab/Istio-Peer-Authentication/assets/157335804/f36822b7-4d0c-4d6a-a72d-e25172cf2f3d)

<br>

### 6) Test Scenario 6

Since we enabled the PeerAuth in Istio-system, all the namespaces have PeerAuth enabled. 

- Inject the Istio in test4 namespace
- Then verify the connection is success.

```yaml

vagrant@istio-cluster:~/kind-demo$ kubectl get po  -n test4
NAME                                   READY   STATUS    RESTARTS   AGE
purchase-history-v1-59c965bb57-sk4z7   2/2     Running   0          117s
recommendation-65f59c68bd-thjrc        2/2     Running   0          93s
sleep-69d4cbb598-wkpbr                 2/2     Running   0          102s
web-api-864b69765c-djjq6               2/2     Running   0          107s

vagrant@istio-cluster:~/kind-demo$ kubectl exec -it sleep-66b45d6984-xr6sc -n istio-in-action -- curl [http://web-api.test4:8080](http://web-api.test4:8080/)
{
"name": "web-api",
"uri": "/",
"type": "HTTP",
"ip_addresses": [
"10.244.2.17"
],
"start_time": "2024-02-26T18:29:16.039891",
"end_time": "2024-02-26T18:29:16.079724",
"duration": "39.833538ms",
"body": "Hello From Web API",
"upstream_calls": [
{
"name": "recommendation",
"uri": "[http://recommendation:8080](http://recommendation:8080/)",
"type": "HTTP",
"ip_addresses": [
"10.244.2.18"
],
"start_time": "2024-02-26T18:29:16.052626",
"end_time": "2024-02-26T18:29:16.068992",
"duration": "16.36569ms",
"body": "Hello From Recommendations!",
"upstream_calls": [
{
"name": "purchase-history-v1",
"uri": "[http://purchase-history:8080](http://purchase-history:8080/)",
"type": "HTTP",
"ip_addresses": [
"10.244.1.15"
],
"start_time": "2024-02-26T18:29:16.060107",
"end_time": "2024-02-26T18:29:16.060324",
"duration": "216.97µs",
"body": "Hello From Purchase History (v1)!",
"code": 200
}
],
"code": 200
}
],
"code": 200
}
```

```yaml
vagrant@istio-cluster:~/kind-demo$ kubectl exec -it sleep-69d4cbb598-wkpbr -n test4 -- curl http://web-api.istio-in-action:8080
{
  "name": "web-api",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "10.244.2.6"
  ],
  "start_time": "2024-02-26T18:30:19.663457",
  "end_time": "2024-02-26T18:30:19.716288",
  "duration": "52.831474ms",
  "body": "Hello From Web API",
  "upstream_calls": [
    {
      "name": "recommendation",
      "uri": "http://recommendation:8080",
      "type": "HTTP",
      "ip_addresses": [
        "10.244.1.4"
      ],
      "start_time": "2024-02-26T18:30:19.680795",
      "end_time": "2024-02-26T18:30:19.707911",
      "duration": "27.116289ms",
      "body": "Hello From Recommendations!",
      "upstream_calls": [
        {
          "name": "purchase-history-v1",
          "uri": "http://purchase-history:8080",
          "type": "HTTP",
          "ip_addresses": [
            "10.244.3.5"
          ],
          "start_time": "2024-02-26T18:30:19.704687",
          "end_time": "2024-02-26T18:30:19.705300",
          "duration": "612.853µs",
          "body": "Hello From Purchase History (v1)!",
          "code": 200
        }
      ],
      "code": 200
    }
  ],
  "code": 200
}
```

![image](https://github.com/myathway-lab/Istio-Peer-Authentication/assets/157335804/b86a48c7-764c-4735-abef-35d0c62eda77)



<br>


### 7) Test Scenario 7


![image](https://github.com/myathway-lab/Istio-Peer-Authentication/assets/157335804/3c7e6994-e1c4-4c1e-8336-2d24c73c5979)

<br>

### 8) Test Scenario 8

![image](https://github.com/myathway-lab/Istio-Peer-Authentication/assets/157335804/df041369-e516-4bbc-a03a-5915c9fffeeb)

<br>

### 9) Test Scenario 9

![image](https://github.com/myathway-lab/Istio-Peer-Authentication/assets/157335804/2814ab4f-3d0f-4d71-b54a-fb815271ffdd)

<br>

### 10) Test Scenario 10
![image](https://github.com/myathway-lab/Istio-Peer-Authentication/assets/157335804/c544495e-1f22-4feb-bec7-531dd40660f7)



### 11) Should we enable PeerAuth in "Istio-System"?

**Benefit**
- Enabling PeerAuth in "Istio-System" will enforce mTLS commnunication and only authorized servers can talk with each other within the Istio service mesh. 

<br>

**Drawback**
- We should plan accordinly since beginning and if we suddenly apply peerauth for "istio-system", some of the services may have istio injected & some may not have. <br>
- And enabling peerauth may disrupt the server functionalities if they cannot use mTLS.

<br>

**Recommendation**
- Start with Plan
- Not recommand to use PeerAuth in "istio-system" namespace.
- We need to verify whether our services are istio ready and can handle mTLS.
- It depends on the organization requirements. 
