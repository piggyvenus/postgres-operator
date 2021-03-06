---
title: "PostgreSQL Operator API Encryption Configuration"
date:
draft: false
weight: 7
---

## Configuring Encryption of PostgreSQL Operator API Connection

The PostgreSQL Operator REST API connection is encrypted with keys stored in the *pgo.tls* Secret.  

The pgo.tls Secret can be generated prior to starting the PostgreSQL Operator or you can let the PostgreSQL Operator generate the Secret for you if the Secret
does not exist.

Adjust the default keys to meet your security requirements using your own keys.  The *pgo.tls* Secret is created when you run:

    make deployoperator

The keys are generated when the RBAC script is executed by the cluster admin:

    make installrbac

In some scenarios like an OLM deployment, it is preferable for the Operator to generate the Secret keys at runtime, if the pgo.tls Secret does not exit when the Operator starts, a new TLS Secret will be generated.

In this scenario, you can extract the generated Secret TLS keys using:

    kubectl cp <pgo-namespace>/<pgo-pod>:/tmp/server.key /tmp/server.key -c apiserver
    kubectl cp <pgo-namespace>/<pgo-pod>:/tmp/server.crt /tmp/server.crt -c apiserver
    
example of the command below:
    
    kubectl cp pgo/postgres-operator-585584f57d-ntwr5:tmp/server.key /tmp/server.key -c apiserver
    kubectl cp pgo/postgres-operator-585584f57d-ntwr5:tmp/server.crt /tmp/server.crt -c apiserver

This server.key and server.crt can then be used to access the *pgo-apiserver* from the pgo CLI by setting the following variables in your client environment:

    export PGO_CA_CERT=/tmp/server.crt
    export PGO_CLIENT_CERT=/tmp/server.crt
    export PGO_CLIENT_KEY=/tmp/server.key

You can view the TLS secret using:

    kubectl get secret pgo.tls -n pgo
or

    oc get secret pgo.tls -n pgo

If you create the Secret outside of the Operator, for example using the default installation script, the key and cert that are generated by the default installation are found here:

    $PGOROOT/conf/postgres-operator/server.crt 
    $PGOROOT/conf/postgres-operator/server.key 

The key and cert are generated using the *deploy/gen-api-keys.sh* script.

That script gets executed when running:

    make installrbac

You can extract the server.key and server.crt from the Secret using the following:

    oc get secret pgo.tls -n $PGO_OPERATOR_NAMESPACE -o jsonpath='{.data.tls\.key}' | base64 --decode > /tmp/server.key
    oc get secret pgo.tls -n $PGO_OPERATOR_NAMESPACE -o jsonpath='{.data.tls\.crt}' | base64 --decode > /tmp/server.crt

This server.key and server.crt can then be used to access the *pgo-apiserver* REST API from the pgo CLI on your client host.
