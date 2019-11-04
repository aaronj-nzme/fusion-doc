Using Environment Variables and "Secrets"
=========================================

Environment variables are exactly what they sound like: variables that are set per environment. They are typically used for things like credentials or domains that may change from local to staging to production environments. In Fusion, environment variables are defined in a `.env` file for the local environment, and in the `/environment/` directory for other environments.

Local vs. production environments
---------------------------------

As mentioned previously, Fusion allows you to set environment variables differently depending on what environment you're in.

In local development, you can define whatever environment variables you want in the `.env` file in the root of your repository. These will override environment variables of the same name defined in the `/environment/` directory. Variables defined in your `.env` file don't need to be encrypted since they are `.gitignore`d and only exist on your local system. So in general, use `.env` for environment variables defined on your local.

In staging and/or production environments you'll define environment variables in the `/environment/` directory. This provides the advantage of keeping them in the repository so you don't need to configure environment variables separately on the server - however, this presents its own problem as many of these variables should be secret and not stored in plaintext in the repository. To alleviate this concern, Fusion has the ability to decrypt secret variables at "deploy" time when provided the correct key.

Creating and using environment variables
----------------------------------------

Let's see how we can actually use environment variables in the sample Fusion app we've been building.

So far, the only environment variable we've used was the `OMDB_API_KEY` in our `movie-find` content source, so let's define that one. The first thing we'll do is go to our `.env` file and add it there:

    #  /.env  #
    
    OMDB_API_KEY=a1b2c3d4e5
    

> **NOTE**
> 
> The `OMDB_API_KEY` value used above is fake - you'll need to get your own from the [OMDB API](https://www.omdbapi.com/) and use that instead!

We now have everything we need for our content source to work locally!

To get it working in staging/production environments, we'll need to first encrypt our secret variable, then create an `index.js` file in our `/environment/` directory.

> **NOTE**
> 
> There are a few different formats you can use for naming the environment variables file. You can define it as a top level file called `/environment.js` or `/environment.json`, or alternatively in the `/environment/` directory as `/environment/index.js` or `/environment/index.json`. Just make sure you only have one of these!
> 
> **NOTE**
> 
> You can also create environment-specific files such as `/environment/bonnier.js` or `/environment/bonnier-sandbox.yml` that will only be used when deployed to that specific environment. The values from these files will be merged with (and take precedence over) those in the generic file above.

#### Encrypting your secrets

To enable secret encryption for your Arc environment, PageBuilder's deployment tool Maestro comes with a simple application to encrypt and decrypt secrets based on your environment. Because the encryption values are based on a shared key that is provisioned per environment, and because Maestro has access to that key, Maestro is able to securely and statelessly encrypt and decrypt secrets for us - and this makes it the perfect tool for getting our Fusion application secrets ready for external environments.

To use the tool, simply go to your environment's [Maestro "Secrets" page](https://redirector.arcpublishing.com/deployments/fusion/secrets). Copy the "secret" value for our `OMDB_API_KEY` and paste it into the "encrypt" text input at the top of the page, then click the "encrypt" button. You should get an encrypted value that looks similar to this:

`AQECAHhPwAyPK3nfERyAvmyWOWx9c41uht+ei4Zlv4NgrlmypwAAAMYwgcMGCSqGSIb3DQEHBqCBtTCBsgIBADCBrAYJKoZIhvcNAQcBMB4GCWCGSAFlAwQBLjARBAxwBJdfzqcQUpox1xsCARCAf2aXwBJ3pBUP12HWB3cdBboV1/qN0HFEsjNycADYIq7XSANeDYOlu2/Dwt/52R16hK4dbVOt0ofNKKx0b3vtZRaH9bX1Dkx6TDhmo5g32H0aWpiUW6PQIp72/g2CW1nr26T0zxmkxmX9u8ufoQGBXRd1pOfT2EliUhMKabNeSyk=`

This encrypted value is now safe to be stored in our git repository and added to our `/environment/index.js` file inside of "percent bracket" control characters, like so:

    /*  /environment/index.js  */
    
    export default {
      OMDB_API_KEY: "%{AQECAHhPwAyPK3nfERyAvmyWOWx9c41uht+ei4Zlv4NgrlmypwAAAMYwgcMGCSqGSIb3DQEHBqCBtTCBsgIBADCBrAYJKoZIhvcNAQcBMB4GCWCGSAFlAwQBLjARBAxwBJdfzqcQUpox1xsCARCAf2aXwBJ3pBUP12HWB3cdBboV1/qN0HFEsjNycADYIq7XSANeDYOlu2/Dwt/52R16hK4dbVOt0ofNKKx0b3vtZRaH9bX1Dkx6TDhmo5g32H0aWpiUW6PQIp72/g2CW1nr26T0zxmkxmX9u8ufoQGBXRd1pOfT2EliUhMKabNeSyk=}"
    }
    

Since this KMS key is assigned on a per-client, per-region basis, Fusion will automatically decrypt this variable at deploy time so it can be used reliably across different environments (Sandbox, Production, etc.) as long as they are in the same region. And while our app would work locally without the value stored in our `/environment/index.js` file (since its decrypted counterpart is already in our `.env` file), adding it now means we won't have to do it later when we deploy our application!

Restrictions
------------

Environment values will only be accessible during server execution, not in the client, as they will be exposed to users. To ensure this, `fusion:environment` will return an empty object in the client.

Now that we have our content source and credentials set up, we can use them to retrieve and display some content!