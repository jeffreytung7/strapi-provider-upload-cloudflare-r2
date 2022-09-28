# strapi-provider-upload-cloudflare-r2

- [LICENSE](LICENSE)
- [Strapi website](https://strapi.io/)
- [Strapi documentation](https://docs.strapi.io)
- [Cloudflare R2 documentation](https://developers.cloudflare.com/r2/)

## Installation

```bash
# using npm
npm install strapi-provider-upload-cloudflare-r2
```
```bash
# using yarn
yarn add strapi-provider-upload-cloudflare-r2
```

## Configuration

- `provider` defines the name of the provider
- `providerOptions` is passed down during the construction of the provider. (ex: `new AWS.S3(config)`). [Complete list of options](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/S3.html#constructor-property)
- `actionOptions` is passed directly to the parameters to each method respectively. You can find the complete list of [upload/ uploadStream options](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/S3.html#upload-property) and [delete options](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/S3.html#deleteObject-property)

See the [documentation about using a provider](https://docs.strapi.io/developer-docs/latest/plugins/upload.html#using-a-provider) for information on installing and using a provider. To understand how environment variables are used in Strapi, please refer to the [documentation about environment variables](https://docs.strapi.io/developer-docs/latest/setup-deployment-guides/configurations/optional/environment.html#environment-variables).

To get the env variables below, [follow these instructions](https://developers.cloudflare.com/r2/platform/s3-compatibility/tokens/).

The R2_ACCESS_KEY_ID and R2_ACCESS_SECRET are given when you make the API token. Give edit access. Note: when creating the token and selecting the time active limit, change to Custom and don't put anything there, that will set it to unlimited time.

The R2_REGION should be set to us-east-1 ([the guide](https://developers.cloudflare.com/r2/platform/s3-compatibility/api/) says auto works but it doesn't).

The R2_BUCKET is the name of the R2 bucket you create, as seen on the Cloudflare R2 dashboard. The R2_ACCOUNT_ID is also on that dashboard. 

The R2_PUBLIC_URL is the full URL of bucket as public which is found in Cloudflare R2 Settings (must include http:// or https://, must not include / at the end of URL. eg: https://pub-b627.r2.dev).

After you load up Strapi and upload an image, it will be publically available at R2_PUBLIC_URL/name_of_the_img_file.png

### Provider Configuration

`./config/plugins.js`

```js
module.exports = ({ env }) => ({
    // ...
    upload: {
        config: {
            provider: 'strapi-provider-upload-cloudflare-r2',
            providerOptions: {
                accessKeyId: env('R2_ACCESS_KEY_ID'),
                secretAccessKey: env('R2_ACCESS_SECRET'),
                region: env('R2_REGION'),
                params: {
                    Bucket: env('R2_BUCKET'),
                    accountId: env('R2_ACCOUNT_ID'),
                    publicUrl: env('R2_PUBLIC_URL'),
                },
            },
            actionOptions: {
                upload: {},
                uploadStream: {},
                delete: {},
            },
        },
    },
    // ...
});
```

### Security Middleware Configuration

Due to the default settings in the Strapi Security Middleware you will need to modify the `contentSecurityPolicy` settings to properly see thumbnail previews in the Media Library. You should replace `strapi::security` string with the object below instead as explained in the [middleware configuration](https://docs.strapi.io/developer-docs/latest/setup-deployment-guides/configurations/required/middlewares.html#loading-order) documentation.

Also be sure to pass the env variable in.

`./config/middlewares.js`

```js
module.exports = ({ env }) => ([
  // ...
  {
    name: 'strapi::security',
    config: {
      contentSecurityPolicy: {
        useDefaults: true,
        directives: {
          'connect-src': ["'self'", 'https:'],
          'img-src': [
            "'self'",
            'data:',
            'blob:',
            'dl.airtable.com',
            env('R2_PUBLIC_URL').replace(/^https?:\/\//, ''), // removes http or https from url
          ],
          'media-src': [
            "'self'",
            'data:',
            'blob:',
            'dl.airtable.com',
            env('R2_PUBLIC_URL').replace(/^https?:\/\//, ''),
          ],
          upgradeInsecureRequests: null,
        },
      },
    },
  },
  // ...
]);
```

