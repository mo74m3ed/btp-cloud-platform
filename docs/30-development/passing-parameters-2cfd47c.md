<!-- loio2cfd47cf52b440e8bad3f0c495db824b -->

# Passing Parameters

You can set input parameters for your resources.



<a name="loio2cfd47cf52b440e8bad3f0c495db824b__section_cf5_bmx_wcc"/>

## Procedure

To pass additional parameters stored outside your resource `spec`, create a Kubernetes Secret manually in the same namespace as your service instance or service binding, and reference it using the `parametersFrom` field. To set input parameters, go to the `spec` of the service instance or service binding resource, and use one or both of the following fields:

-   `parameters`: Specifies a set of properties sent to the service broker. The specified data is passed to the service broker without any modifications aside from converting it to JSON for transmission to the broker if the `spec` field is specified as YAML. All valid YAML or JSON constructs are supported.

-   `parametersFrom`: Specifies which Secret, together with the key in it, to include in the set of parameters sent to the service broker. The key contains a `string` that represents a JSON file. The `parametersFrom` field is a list that supports multiple sources referenced per `spec`. The service instance resource can specify multiple related Secrets.


For service instance resources, you can also use the following parameter:

-   `watchParametersFromChanges`: Use this field together with `parametersFrom`. This field is only relevant for service instance resources because you cannot update service binding resources. Set it to `true` to trigger an automatic update of the service instance resource with the changes to the Secret values listed in `parametersFrom`. By default, it is set to `false`.


> ### Caution:  
> When using `watchParametersFromChanges`, the referenced Secret must have the label `services.cloud.sap.com/managed-by-sap-btp-operator: "true"`. Without the label, the SAP BTP service operator does not watch the Secret, and changes to it are not detected.

If you specify multiple sources in the `parameters` and `parametersFrom` fields, the final payload merges all of them at the top level.

To avoid errors, do not use the same top-level parameter name in multiple sources in the `parameters` and `parametersFrom` fields. Otherwise, the specification is invalid, and further processing of the service instance or service binding resources stops with the status ***Error***.



<a name="loio2cfd47cf52b440e8bad3f0c495db824b__section_uxh_dnx_wcc"/>

## Examples

The following example shows a service instance `spec` that uses `parameters`, `parametersFrom`, and `watchParametersFromChanges`:

```
spec:
  ...
  parameters:
    name: value
  parametersFrom:
    - secretKeyRef:
        name: {SECRET_NAME}
        key: secret-parameter
  watchParametersFromChanges: true      
```

You must create the Secret referenced by `parametersFrom` in the same namespace as the service instance:

```
apiVersion: v1
kind: Secret
metadata:
  name: {SECRET_NAME}
  namespace: {NAMESPACE}
  labels:
    services.cloud.sap.com/managed-by-sap-btp-operator: "true"
type: Opaque
stringData:
  secret-parameter:
    '{
      "password": "password",
      "key2": "value2",
      "key3": "value3"
    }'
```

The values from `parameters` and `parametersFrom` are merged into a single JSON payload sent to the service broker:

```
{
  "name": "value",
  "password": "password",
  "key2": "value2",
  "key3": "value3"
}
```

