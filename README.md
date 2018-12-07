# Summary
Provides the ability to externalize the secure values of your auth0-cli-deploy configuration file to AWS Secret manager. This way they are not stored in source control where the value can be accidentally leaked.


# Usage
1. Install `npm i -g auth0-deploy-cli-config-values aws-sdk`
1. Setup AWS `aws configure`. You can alternatively edit your .credentials file directly. The aws-sdk in use within this module will pickup the values appropriately. This includes being able to specify a profile at execution time. Make sure the account/profile you are using has permissions to read the secrets, see **Permissions** below for more details.
1. Update your configuration file for auth0 deploy CLI. Each value you want replaced must be in the format of secret:{aws secret manager path}:{key within secrets}. See more on this below.
1. Execute auth0-deploy-cli-config-values to modify your configuration file, before it is used. `a0deploy-config-values -p dev -c a0deploy.cfg.json`. Or the long form of `a0deploy-config-values --aws-profile dev --a0deploy-config a0deploy.cfg.json`.


# Double Hash
Before secrets are resolved, the file has all of its double hash values resolved. Within your secret values you can use double hash to distinguish which environment the configuration will be for. The below example shows you an example configuration file were the environment is used to choose a different secret bundle based on the environment.

The logic will also respect nested values, so you can chain them as appropriate.

```
{
  "AUTH0_DOMAIN": "YOUR_DOMAIN.auth0.com",
  "AUTH0_CLIENT_ID": "secret:/##STAGE##/auth0Deploy:CLIENT_ID",
  "AUTH0_CLIENT_SECRET": "secret:/##STAGE##/auth0Deploy:CLIENT_ID",
  "AUTH0_ALLOW_DELETE": true,
  "AUTH0_KEYWORD_REPLACE_MAPPINGS": {
    "ENVIRONMENT": "dev",
    "STAGE": "##ENVIRONMENT##-PR102",
    "APP1_SECRET": "secret:/##STAGE##/app1:SUPER_SECRET",
    "APP2_SECRET": "secret:/##STAGE##/app2:SUPER_SECRET"
  }
}
```

# AWS Secret Lookup
Secrets will be added to values of the configurations by looking for `secret:` at the begining of a value. Next, the AWS Secret to lookup will be determined between the two semi-colons. Next, the final value will be looked up in the secret file. If nothing is found, an exception is thrown.

For instance, if you have a secret file called /dev/auth0Secrets and its contents are `{"FOO":"bar"}`. To inject this value into your configuraiton file you would specify secret:/dev/auth0Secrets:FOO. This functionallity respects the double hash notation of auth0-deploy-cli, see the **Double Hash** section for more details.
```
{
  "AUTH0_DOMAIN": "YOUR_DOMAIN.auth0.com",
  "AUTH0_CLIENT_ID": "secret:/##STAGE##/auth0Deploy:CLIENT_ID",
  "AUTH0_CLIENT_SECRET": "secret:/##STAGE##/auth0Deploy:CLIENT_ID",
  "AUTH0_ALLOW_DELETE": true,
  "AUTH0_KEYWORD_REPLACE_MAPPINGS": {
    "ENVIRONMENT": "dev",
    "STAGE": "##ENVIRONMENT##-PR102",
    "APP1_SECRET": "secret:/##STAGE##/app1:SUPER_SECRET",
    "APP2_SECRET": "secret:/##STAGE##/app2:SUPER_SECRET"
  }
}
```

# Permissions
-  Effect: "Allow"
       Action:
          - "secretsmanager:DescribeSecret"
          - "secretsmanager:GetSecretValue"
       Resource:
          - arn:aws:secretsmanager:#{AWS::Region}:#{AWS::AccountId}:secret:/${self:provider.stage}/users*

# CLI Arguments
1. --help to get a general review on how to use this tool.
1. --config or -c, (required) to specify which configuration file to read and update.
1. --output or -o, (optiona) specifies the new file to create. If omitted, the configuration file will be updated directly.
1. --aws-profile or -p, (optional) to specify which AWS profile to use when accessing the secrets.
1. --aws-region or -r, (optiona) the region to look for the secrets in, if your profile or environment variables do not specify a region.