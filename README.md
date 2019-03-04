# fastly-native-promises

> Native Promise based Fastly API client for Node.js

[![NPM Version](https://img.shields.io/npm/v/@adobe/fastly-native-promises.svg)](https://www.npmjs.com/package/@adobe/fastly-native-promises)
[![Known Vulnerabilities](https://snyk.io/test/github/adobe/fastly-native-promises/badge.svg?targetFile=package.json)](https://snyk.io/test/github/adobe/fastly-native-promises?targetFile=package.json)
[![codecov](https://img.shields.io/codecov/c/github/adobe/fastly-native-promises.svg)](https://codecov.io/gh/adobe/fastly-native-promises)
[![CircleCI](https://img.shields.io/circleci/project/github/adobe/fastly-native-promises.svg)](https://circleci.com/gh/adobe/fastly-native-promises)
[![GitHub license](https://img.shields.io/github/license/adobe/fastly-native-promises.svg)](https://github.com/adobe/fastly-native-promises/blob/master/LICENSE)
[![GitHub issues](https://img.shields.io/github/issues/adobe/fastly-native-promises.svg)](https://github.com/adobe/fastly-native-promises/issues) [![Greenkeeper badge](https://badges.greenkeeper.io/adobe/fastly-native-promises.svg)](https://greenkeeper.io/)

## Problem

The callback based [fastly](https://www.npmjs.com/package/fastly) package is still the most used client on [NPM](https://www.npmjs.com/). However, I needed a client which allows me to perform request sequentially and parallelly without ending up in an untamable [callback hell](http://callbackhell.com/). [Philipp Schulte's fastly-native-promises](https://github.com/philippschulte/fastly-native-promises) client seemed almost perfect, except:

- it uses Axios, which is an additional dependency we'd like to avoid, especially when running inside Adobe I/O Runtime
- it has been missing features and pull requests were merged only slowly

This fork addresses the concerns above, but breaks compatibility with Browsers, so that it can only be used in Node JS environments.

## Solution

The `fastly-native-promises` package uses the promise based HTTP client [Request-Promise-Native](https://github.com/request/request-promise-native) to perform requests to the [Fastly](https://docs.fastly.com/api/) API. [Request-Promise-Native](https://github.com/request/request-promise-native) supports the native JavaScript [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) API and automatically transforms the data into JSON. Each `fastly-native-promises` API method returns a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) which represents either the completion or failure of the request. 

## Table of Contents

- [Security](#security)
- [Install](#install)
- [Usage](#usage)
- [API](#api)
- [Tests](#tests)
- [Contribute](#contribute)
- [License](#license)

## Security

You'll need a [Fastly API Token](https://docs.fastly.com/api/auth#tokens) in order to use the `fastly-native-promises` library. I recommend to use a token with a [global scope](https://docs.fastly.com/api/auth#access) to be able to use all `fastly-native-promises` API methods.

## Install

This is a [Node.js](https://nodejs.org/) module available through the [npm registry](https://www.npmjs.com/). Installation is done using the [`npm install` command](https://docs.npmjs.com/getting-started/installing-npm-packages-locally):

```bash
$ npm install @adobe/fastly-native-promises
```

## Changes

See the [changelog](CHANGELOG.md).

## Usage

```javascript
const fastly = require('@adobe/fastly-native-promises');

// create one or more instances
const service_1 = fastly('token', 'service_id_1');
const serivce_2 = fastly('token', 'service_id_2');

// make changes

service_1.transact(async () => {
  return this.writeS3('test-s3', {
    name: 'test-s3',
    bucket_name: 'my_corporate_bucket',
    access_key: 'AKIAIOSFODNN7EXAMPLE',
    secret_key: 'wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY',
  });
});


service_2.transact(async () => {
  return this.writeBigquery('test-bq', {
    name: 'test-bq',
    format: '{\n "timestamp":"%{begin:%Y-%m-%dT%H:%M:%S}t",\n  "time_elapsed":%{time.elapsed.usec}V,\n  "is_tls":%{if(req.is_ssl, "true", "false")}V,\n  "client_ip":"%{req.http.Fastly-Client-IP}V",\n  "geo_city":"%{client.geo.city}V",\n  "geo_country_code":"%{client.geo.country_code}V",\n  "request":"%{req.request}V",\n  "host":"%{req.http.Fastly-Orig-Host}V",\n  "url":"%{json.escape(req.url)}V",\n  "request_referer":"%{json.escape(req.http.Referer)}V",\n  "request_user_agent":"%{json.escape(req.http.User-Agent)}V",\n  "request_accept_language":"%{json.escape(req.http.Accept-Language)}V",\n  "request_accept_charset":"%{json.escape(req.http.Accept-Charset)}V",\n  "cache_status":"%{regsub(fastly_info.state, "^(HIT-(SYNTH)|(HITPASS|HIT|MISS|PASS|ERROR|PIPE)).*", "\\\\2\\\\3") }V"\n}',
    user: 'fastly-bigquery-log@example-fastly-log.iam.gserviceaccount.com',
    project_id: 'example-fastly-log',
    dataset: 'fastly_log_test',
    table: 'fastly_logs',
    template_suffix: null,
    secret_key: '-----BEGIN PRIVATE KEY-----\nMIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQC7bPG9yaIYd5AL\nmvOaYvNozFJB/VWS53KWBll769kJvlmgMks6r6Xrv8w6rjxWKjZeDrnXVf7UDa0F\nckPPIFvXRxahftWFMGArw0lIvQzgT4/BlndXU5RNxfah/8m7q/GIF6oNYWzfJwvv\nzodxDUqIRH2e2JWidNRjElHuogYHLhV4O/od5pAkfDwak/ihuuh/2VA3Auwb3nph\ndX2F0JBs14oPKZUTYUUSzUQY5IMxSxYUA4Q7W4v21x1EnJt+biXOrERk1rm4ieEE\nU3WkjR5c5gvG8xcWyYod87RNFELmIhCCytI1+t5C3Em/jPsQFtLzwHpbNhdW4oEm\nn7d06n75AgMBAAECggEAWRh26lNZfOwJS5sDRlbXgu/uAnSdI1JmxC6Mhz4cVGdq\nT57Y6DLrWuA4A4UkJYm3gorZiSXWF5PQthAVb/bf8bxXY7nZYpEWhnc09SD5aAAq\nREp0vMx8aWQ709K2YUJg+zDUo7u2d3YmVH8HH5TD43c7iDFJIIsNE3N4A0p+NxZ+\nw06FFW+fz/etrWiNyhrlTsbkMbSgU+GpFFBq1pCd0ni5d1YM1rsaAaUpmkwdjgjL\noDs+M/L/HtqfEhyZNdw8JF7EJXVE1bIl7/NL0rBInhyO28FcB56t/AG5nzXKFI/c\nc+IO7d6MOOqiGRLRWZItEpnyzuV8DZo461wy1hSvqQKBgQDhSsg2cHkTrtBW8x0A\n3BwB/ygdkkxm1OIvfioT+JBneRufUPvVIM2aPZBBGKEedDAmIGn/8f9XAHhKjs8B\nEsPRgE206s4+hnrTcK7AeWWPvM9FDkrkQCoJFuJrNy9mJt8gs7AnnoBa9u/J4naW\ne1tfC8fUfsa7kdzblDhcRQ8FhwKBgQDU+N4kPzIdUuJDadd6TkBbjUNPEfZzU5+t\nIike2VSRhApxAxviUnTDsTROwJRzKik9w7gIMka8Ek+nmLNMEtds77ttcGQRdu16\n+vT1iualiCJe+/iMbl+PiJtFwhEHECLU9QfgBVS6r2lDAlZA+w6nwCRiidlrObzO\nCXqVOzN3fwKBgAsrOuu//bClHP0ChnCReO38aU+1/gWnDiOOnKVq0DXhAiaOzD1P\nqAG6hZlEkFBDMPWzq62doKv+gPgpRkfmV0DenHuYnGrrHdG3p2IxYoCSuq/QupPA\nPpU+xjDMhpQI30zuu4/rQq+/yDl4+aoSKYB3xAtb0Zxg6dMU8QpZ/hmnAoGBAIFu\nIesbcQR7O8FGkMrmxZweNNrYCtQ57R/WU/FImWm6OnJGNmsMO6Q2jJiT12RKKjg8\nOxrYGz7vTfOIDOddyAiPhXPUSyyF/3uvCrIzUUsmeeUJ8xq9dVwQ5HS3pYuKVfDg\nXYHbG4w9UJaF1A+3xEdUsYglSLouo7z/67zH9tZXAoGBAKpsdjSd3R+llaAv2HQ8\nGMlN92UTr5i9w++QMXq4qspH5NEYqz3NHbKuYthZqxEsRUZbRP50eDWU4jvxFVJl\nLBFINp6B+3AsIme0YCyOaleB/Cy0347miSinSv2I6QiH6dQxHdHzrG+x1evS/76f\nKT0KS+ySjCAEWgg4v+mjUDUV\n-----END PRIVATE KEY-----\n',
    response_condition: '',
  });
});
```

### Promises

Purge all domains of the active version:

1. Get all the versions
2. Filter out the active version
3. Get all the domains for the active version
4. Purge all the domains
5. Log the status text for each purge request

```javascript
const fastly = require('fastly-native-promises');

const service = fastly('token', 'service_id');

function handler() {
  service.readVersions()
    .then(versions => {
      const active = versions.data.filter(version => version.active)[0];
      return service.readDomains(active.number);
    })
    .then(domains => {
      return Promise.all(domains.data.map(domain => service.purgeIndividual(domain.name)));
    })
    .then(purges => {
      purges.forEach(purge => console.log(purge.statusText));
    })
    .catch(e => {
      console.log('Shoot!');
    });
}
```

### Async/Await

Update `first_byte_timeout` property for every backend and service if the value is less than 5000 milliseconds:

1.  Get all the services associated with the Fastly API token
2.  Filter out the service IDs
3.  Iterate over all services synchronously
4.  Get all the versions
5.  Filter out the active version
6.  Get all the backends for the active version
7.  Filter out the affected backends
8.  Continue with the next service if there are no affected backends
9.  Clone the active version
10. Update all the affected backends parallelly
11. Activate the cloned version

```javascript
const fastly = require('fastly-native-promises');

const account = fastly('token');

async function handler() {
  try {
    const services = await account.readServices();
    const ids = services.data.map(service => service.id);

    for (const id of ids) {
      const service = fastly('token', id);
      const versions = await service.readVersions();
      const active = versions.data.filter(version => version.active)[0];
      const backends = await service.readBackends(active.number);
      const affected = backends.data.filter(backend => backend.first_byte_timeout < 5000);

      if (!affected.length) continue;

      const clone = await service.cloneVersion(active.number);
      await Promise.all(affected.map(backend => service.updateBackend(clone.data.number, backend.name, { first_byte_timeout: 5000 })));
      await service.activateVersion(clone.data.number);
    }
  } catch (e) {
    console.log('Shoot!');
  }
}
```

### Response Schema

Each `fastly-native-promises` API method returns the following response object:

```javascript
{
  // the HTTP status code from the server response
  status: 200,

  // the HTTP status message from the server response
  statusText: 'OK',

  // the headers that the server responded with
  headers: {},

  // the options that were provided to request for the request
  config: {},

  // the request that generated the response
  request: {},

  // the response that was provided by the server
  data: {}
}
```

## API

### Classes

<dl>
<dt><a href="#Fastly">Fastly</a></dt>
<dd></dd>
</dl>

### Typedefs

<dl>
<dt><a href="#CreateFunction">CreateFunction</a> ⇒ <code>Promise</code></dt>
<dd><p>A function that creates a resource of a specific type. If a resource of that
name already exists, it will reject the returned promise with an error.</p>
</dd>
<dt><a href="#UpdateFunction">UpdateFunction</a> ⇒ <code>Promise</code></dt>
<dd><p>A function that updates an already existing resource of a specific type.
If no resource of that name exists, it will reject the returned promise with an error.</p>
</dd>
<dt><a href="#ReadFunction">ReadFunction</a> ⇒ <code>Promise</code></dt>
<dd><p>A function that retrieves a representation of a resource of a specific type.
If no resource of that name exists, it will reject the returned promise with an error.</p>
</dd>
<dt><a href="#ListFunction">ListFunction</a> ⇒ <code>Promise</code></dt>
<dd><p>A function that retrieves a list of resources of a specific type.</p>
</dd>
<dt><a href="#FastlyError">FastlyError</a> : <code>Object</code></dt>
<dd><p>The FastlyError class describes the most common errors that can occur
when working with the Fastly API. Using <code>error.status</code>, the underlying
HTTP status code can be retrieved. Known error status codes include:</p>
<ul>
<li>400: attempting to activate invalid VCL</li>
<li>401: invalid credentials</li>
<li>404: resource not found</li>
<li>409: confict when trying to POST a resource that already exists</li>
<li>422: attempting to modify a service config that is not checked out</li>
<li>429: rate limit exceeded, try again later</li>
</ul>
</dd>
<dt><a href="#Response">Response</a> : <code>Object</code></dt>
<dd></dd>
<dt><a href="#Versions">Versions</a> : <code>Object</code></dt>
<dd><p>Describes the most relevant versions of the service.</p>
</dd>
<dt><a href="#DictUpdate">DictUpdate</a> : <code>Object</code></dt>
<dd></dd>
<dt><a href="#Snippet">Snippet</a> : <code>Object</code></dt>
<dd></dd>
<dt><a href="#VCL">VCL</a> : <code>Object</code></dt>
<dd></dd>
</dl>

<a name="Fastly"></a>

### Fastly
**Kind**: global class  

* [Fastly](#Fastly)
    * [new Fastly(token, service_id)](#new_Fastly_new)
    * [.readLogsFn(service)](#Fastly+readLogsFn) ⇒ [<code>ListFunction</code>](#ListFunction)
    * [.readLogFn(service)](#Fastly+readLogFn) ⇒ [<code>ReadFunction</code>](#ReadFunction)
    * [.createLogFn(service)](#Fastly+createLogFn) ⇒ [<code>CreateFunction</code>](#CreateFunction)
    * [.updateLogFn(service)](#Fastly+updateLogFn) ⇒ [<code>UpdateFunction</code>](#UpdateFunction)
    * [.upsertFn(createFn, updateFn, readFn)](#Fastly+upsertFn) ⇒ [<code>UpdateFunction</code>](#UpdateFunction)
    * [.purgeIndividual(url)](#Fastly+purgeIndividual) ⇒ <code>Promise</code>
    * [.purgeAll()](#Fastly+purgeAll) ⇒ <code>Promise</code>
    * [.purgeKey(key)](#Fastly+purgeKey) ⇒ <code>Promise</code>
    * [.purgeKeys(keys)](#Fastly+purgeKeys) ⇒ <code>Promise</code>
    * [.softPurgeIndividual(url)](#Fastly+softPurgeIndividual) ⇒ <code>Promise</code>
    * [.softPurgeKey(key)](#Fastly+softPurgeKey) ⇒ <code>Promise</code>
    * [.dataCenters()](#Fastly+dataCenters) ⇒ <code>Promise</code>
    * [.publicIpList()](#Fastly+publicIpList) ⇒ <code>Promise</code>
    * [.edgeCheck(url)](#Fastly+edgeCheck) ⇒ <code>Promise</code>
    * [.readServices()](#Fastly+readServices) ⇒ <code>Promise</code>
    * [.readVersions()](#Fastly+readVersions) ⇒ <code>Promise</code>
    * [.getVersions()](#Fastly+getVersions) ⇒ [<code>Versions</code>](#Versions)
    * [.cloneVersion(version)](#Fastly+cloneVersion) ⇒ <code>Promise</code>
    * [.activateVersion(version)](#Fastly+activateVersion) ⇒ <code>Promise</code>
    * [.domainCheckAll(version)](#Fastly+domainCheckAll) ⇒ <code>Promise</code>
    * [.readDomains(version)](#Fastly+readDomains) ⇒ <code>Promise</code>
    * [.readDictItems(version, name)](#Fastly+readDictItems) ⇒ <code>Promise</code>
    * [.readDictItem(version, name, key)](#Fastly+readDictItem) ⇒ <code>Promise</code>
    * [.createDictItem(version, name, key, value)](#Fastly+createDictItem) ⇒ <code>Promise</code>
    * [.bulkUpdateDictItems(version, name, ...items)](#Fastly+bulkUpdateDictItems) ⇒ <code>Promise</code>
    * [.updateDictItem(version, name, key, value)](#Fastly+updateDictItem) ⇒ <code>Promise</code>
    * [.deleteDictItem(version, name, key)](#Fastly+deleteDictItem) ⇒ <code>Promise</code>
    * [.writeDictItem(version, name, key, value)](#Fastly+writeDictItem) ⇒ <code>Promise</code>
    * [.readDictionaries(version)](#Fastly+readDictionaries) ⇒ <code>Promise</code>
    * [.readDictionary(version, name)](#Fastly+readDictionary) ⇒ <code>Promise</code>
    * [.createDictionary(version, data)](#Fastly+createDictionary) ⇒ <code>Promise</code>
    * [.updateDictionary(version, name, data)](#Fastly+updateDictionary) ⇒ <code>Promise</code>
    * [.deleteDictionary(version, name)](#Fastly+deleteDictionary) ⇒ <code>Promise</code>
    * [.readBackends(version)](#Fastly+readBackends) ⇒ <code>Promise</code>
    * [.updateBackend(version, name, data)](#Fastly+updateBackend) ⇒ <code>Promise</code>
    * [.createBackend(version, data)](#Fastly+createBackend) ⇒ <code>Promise</code>
    * [.createSnippet(version, data)](#Fastly+createSnippet) ⇒ <code>Promise</code>
    * [.updateSnippet(version, name, data)](#Fastly+updateSnippet) ⇒ <code>Promise</code>
    * [.createVCL(version, data)](#Fastly+createVCL) ⇒ <code>Promise</code>
    * [.updateVCL(version, name, data)](#Fastly+updateVCL) ⇒ <code>Promise</code>
    * [.setMainVCL(version, name)](#Fastly+setMainVCL) ⇒ <code>Promise</code>
    * [.transact(operations, activate)](#Fastly+transact) ⇒ <code>Object</code>
    * [.dryrun(operations)](#Fastly+dryrun) ⇒ <code>Object</code>

<a name="new_Fastly_new"></a>

#### new Fastly(token, service_id)
The constructor method for creating a fastly-promises instance.


| Param | Type | Description |
| --- | --- | --- |
| token | <code>string</code> | The Fastly API token. |
| service_id | <code>string</code> | The Fastly service ID. |

<a name="Fastly+readLogsFn"></a>

#### fastly.readLogsFn(service) ⇒ [<code>ListFunction</code>](#ListFunction)
Create a new function that lists all log configurations for a given service
and version. The function can be parametrized with the name of the logging
service.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: [<code>ListFunction</code>](#ListFunction) - A logging function.  

| Param | Type | Description |
| --- | --- | --- |
| service | <code>string</code> | The id of the logging service. Supported services are: s3, s3canary, azureblob, cloudfiles, digitalocean, ftp, bigquery, gcs, honeycomb, logshuttle, logentries, loggly, heroku, openstack, papertrail, scalyr, splunk, sumologic, syslog. |

<a name="Fastly+readLogFn"></a>

#### fastly.readLogFn(service) ⇒ [<code>ReadFunction</code>](#ReadFunction)
Create a new function that returns a named log configuration for a given service
and version. The function can be parametrized with the name of the logging
service.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: [<code>ReadFunction</code>](#ReadFunction) - A logging function.  

| Param | Type | Description |
| --- | --- | --- |
| service | <code>string</code> | The id of the logging service. Supported services are: s3, s3canary, azureblob, cloudfiles, digitalocean, ftp, bigquery, gcs, honeycomb, logshuttle, logentries, loggly, heroku, openstack, papertrail, scalyr, splunk, sumologic, syslog. |

<a name="Fastly+createLogFn"></a>

#### fastly.createLogFn(service) ⇒ [<code>CreateFunction</code>](#CreateFunction)
Create a new function that creates a named log configuration for a given service
and version. The function can be parametrized with the name of the logging
service.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: [<code>CreateFunction</code>](#CreateFunction) - A logging function.  

| Param | Type | Description |
| --- | --- | --- |
| service | <code>string</code> | The id of the logging service. Supported services are: s3, s3canary, azureblob, cloudfiles, digitalocean, ftp, bigquery, gcs, honeycomb, logshuttle, logentries, loggly, heroku, openstack, papertrail, scalyr, splunk, sumologic, syslog. |

<a name="Fastly+updateLogFn"></a>

#### fastly.updateLogFn(service) ⇒ [<code>UpdateFunction</code>](#UpdateFunction)
Create a new function that updates a named log configuration for a given service
and version. The function can be parametrized with the name of the logging
service.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: [<code>UpdateFunction</code>](#UpdateFunction) - A logging function.  

| Param | Type | Description |
| --- | --- | --- |
| service | <code>string</code> | The id of the logging service. Supported services are: s3, s3canary, azureblob, cloudfiles, digitalocean, ftp, bigquery, gcs, honeycomb, logshuttle, logentries, loggly, heroku, openstack, papertrail, scalyr, splunk, sumologic, syslog. |

<a name="Fastly+upsertFn"></a>

#### fastly.upsertFn(createFn, updateFn, readFn) ⇒ [<code>UpdateFunction</code>](#UpdateFunction)
Creates an update-or-create or "safe create" function that will either create
(if it does not exist) or update (if it does) a named resource. The function
will attempt to check if the resource exists first (if a reader function has been
provided), alternatively, it will just blindly create and fall back with an
update.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: [<code>UpdateFunction</code>](#UpdateFunction) - An update function that does not fail on conflict.  

| Param | Type | Description |
| --- | --- | --- |
| createFn | [<code>CreateFunction</code>](#CreateFunction) | A function that creates a resource. |
| updateFn | [<code>UpdateFunction</code>](#UpdateFunction) | A function that updates a resource. |
| readFn | [<code>ReadFunction</code>](#ReadFunction) | An optional function that checks for the existence of a resource. |

<a name="Fastly+purgeIndividual"></a>

#### fastly.purgeIndividual(url) ⇒ <code>Promise</code>
Instant Purge an individual URL.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Fulfil**: [<code>Response</code>](#Response)  
**See**: https://docs.fastly.com/api/purge#purge_3aa1d66ee81dbfed0b03deed0fa16a9a  

| Param | Type | Description |
| --- | --- | --- |
| url | <code>string</code> | The URL to purge. |

**Example**  
```js
instance.purgeIndividual('www.example.com')
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+purgeAll"></a>

#### fastly.purgeAll() ⇒ <code>Promise</code>
Instant Purge everything from a service.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Fulfil**: [<code>Response</code>](#Response)  
**Reject**: [<code>FastlyError</code>](#FastlyError)  
**See**: https://docs.fastly.com/api/purge#purge_bee5ed1a0cfd541e8b9f970a44718546  
**Example**  
```js
instance.purgeAll()
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+purgeKey"></a>

#### fastly.purgeKey(key) ⇒ <code>Promise</code>
Instant Purge a particular service of items tagged with a Surrogate Key.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Fulfil**: [<code>Response</code>](#Response)  
**See**: https://docs.fastly.com/api/purge#purge_d8b8e8be84c350dd92492453a3df3230  

| Param | Type | Description |
| --- | --- | --- |
| key | <code>string</code> | The surrogate key to purge. |

**Example**  
```js
instance.purgeKey('key_1')
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+purgeKeys"></a>

#### fastly.purgeKeys(keys) ⇒ <code>Promise</code>
Instant Purge a particular service of items tagged with Surrogate Keys in a batch.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**See**: https://docs.fastly.com/api/purge#purge_db35b293f8a724717fcf25628d713583  

| Param | Type | Description |
| --- | --- | --- |
| keys | <code>Array</code> | The array of surrogate keys to purge. |

**Example**  
```js
instance.purgeKeys(['key_2', 'key_3', 'key_4'])
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+softPurgeIndividual"></a>

#### fastly.softPurgeIndividual(url) ⇒ <code>Promise</code>
Soft Purge an individual URL.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Fulfil**: [<code>Response</code>](#Response)  
**See**: https://docs.fastly.com/api/purge#soft_purge_0c4f56f3d68e9bed44fb8b638b78ea36  

| Param | Type | Description |
| --- | --- | --- |
| url | <code>string</code> | The URL to soft purge. |

**Example**  
```js
instance.softPurgeIndividual('www.example.com/images')
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+softPurgeKey"></a>

#### fastly.softPurgeKey(key) ⇒ <code>Promise</code>
Soft Purge a particular service of items tagged with a Surrogate Key.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**See**: https://docs.fastly.com/api/purge#soft_purge_2e4d29085640127739f8467f27a5b549  

| Param | Type | Description |
| --- | --- | --- |
| key | <code>string</code> | The surrogate key to soft purge. |

**Example**  
```js
instance.softPurgeKey('key_5')
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+dataCenters"></a>

#### fastly.dataCenters() ⇒ <code>Promise</code>
Get a list of all Fastly datacenters.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Fulfil**: [<code>Response</code>](#Response)  
**See**: https://docs.fastly.com/api/tools#datacenter_1c8d3b9dd035e301155b44eae05e0554  
**Example**  
```js
instance.dataCenters()
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+publicIpList"></a>

#### fastly.publicIpList() ⇒ <code>Promise</code>
Fastly's services IP ranges.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**See**: https://docs.fastly.com/api/tools#public_ip_list_ef2e9900a1c9522b58f5abed92ec785e  
**Example**  
```js
instance.publicIpList()
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+edgeCheck"></a>

#### fastly.edgeCheck(url) ⇒ <code>Promise</code>
Retrieve headers and MD5 hash of the content for a particular URL from each Fastly edge server.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Fulfil**: [<code>Response</code>](#Response)  
**See**: https://docs.fastly.com/api/tools#content_4d2d4548b29c7661e17ebe7098872d6d  

| Param | Type | Description |
| --- | --- | --- |
| url | <code>string</code> | Full URL (host and path) to check on all nodes. If protocol is omitted,    http will be assumed. |

**Example**  
```js
instance.edgeCheck('api.example.com')
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+readServices"></a>

#### fastly.readServices() ⇒ <code>Promise</code>
List all services.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**See**: https://docs.fastly.com/api/config#service_74d98f7e5d018256e44d1cf820388ef8  
**Example**  
```js
instance.readServices()
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+readVersions"></a>

#### fastly.readVersions() ⇒ <code>Promise</code>
List the versions for a particular service.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Fulfil**: [<code>Response</code>](#Response)  
**See**: https://docs.fastly.com/api/config#version_dfde9093f4eb0aa2497bbfd1d9415987  
**Example**  
```js
instance.readVersions()
   .then(res => {
     const active = res.data.filter(version => version.active);
     console.log(active);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+getVersions"></a>

#### fastly.getVersions() ⇒ [<code>Versions</code>](#Versions)
Gets the version footprint for the service.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: [<code>Versions</code>](#Versions) - The latest, current, and active versions of the service.  
<a name="Fastly+cloneVersion"></a>

#### fastly.cloneVersion(version) ⇒ <code>Promise</code>
Clone the current configuration into a new version.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**See**: https://docs.fastly.com/api/config#version_7f4937d0663a27fbb765820d4c76c709  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The version to be cloned. |

**Example**  
```js
instance.cloneVersion('45')
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+activateVersion"></a>

#### fastly.activateVersion(version) ⇒ <code>Promise</code>
Activate the current version.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Fulfil**: [<code>Response</code>](#Response)  
**See**: https://docs.fastly.com/api/config#version_0b79ae1ba6aee61d64cc4d43fed1e0d5  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The version to be activated. |

**Example**  
```js
instance.activateVersion('23')
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+domainCheckAll"></a>

#### fastly.domainCheckAll(version) ⇒ <code>Promise</code>
Checks the status of all domains for a particular service and version.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**See**: https://docs.fastly.com/api/config#domain_e33a599694c3316f00b6b8d53a2db7d9  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The current version of a service. |

**Example**  
```js
instance.domainCheckAll('182')
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+readDomains"></a>

#### fastly.readDomains(version) ⇒ <code>Promise</code>
List all the domains for a particular service and version.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Fulfil**: [<code>Response</code>](#Response)  
**See**: https://docs.fastly.com/api/config#domain_6d340186666771f022ca20f81609d03d  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The current version of a service. |

**Example**  
```js
instance.readDomains('182')
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+readDictItems"></a>

#### fastly.readDictItems(version, name) ⇒ <code>Promise</code>
List all dictionary items for a particular service and version.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**See**: https://docs.fastly.com/api/config#dictionary_item_a48de28cd7e76c1ea58523f39bb7204b  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The version of the dictionary. |
| name | <code>string</code> | The name of the dictionary. |

**Example**  
```js
instance.readDictItems(1, 'my_dictionary')
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+readDictItem"></a>

#### fastly.readDictItem(version, name, key) ⇒ <code>Promise</code>
Get details of a single dictionary item.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**See**: https://docs.fastly.com/api/config#dictionary_item_08f090cd03ed4602ae63f131087e2f29  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The current version of a service. |
| name | <code>string</code> | Name of the dictionary. |
| key | <code>string</code> | The key to retrieve values by. |

**Example**  
```js
instance.readDictItem('12', 'extensions', 'some_key')
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+createDictItem"></a>

#### fastly.createDictItem(version, name, key, value) ⇒ <code>Promise</code>
Create a new dictionary item for a particular service and version.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The reponse object.  
**Fulfil**: [<code>Response</code>](#Response)  
**See**: https://docs.fastly.com/api/config#dictionary_item_6ec455c0ba1b21671789e1362bc7fe55  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>number</code> | The version number (current if omitted). |
| name | <code>Object</code> | The dictionary definition. |
| key | <code>string</code> | The key. |
| value | <code>string</code> | The value to write. |

<a name="Fastly+bulkUpdateDictItems"></a>

#### fastly.bulkUpdateDictItems(version, name, ...items) ⇒ <code>Promise</code>
Updates multiple dictionary items in bulk.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object.  
**Fulfil**: [<code>Response</code>](#Response)  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>number</code> | The version numer (current if ommitted). |
| name | <code>string</code> | Name of the dictionary. |
| ...items | [<code>DictUpdate</code>](#DictUpdate) | The dictionary update operations. |

<a name="Fastly+updateDictItem"></a>

#### fastly.updateDictItem(version, name, key, value) ⇒ <code>Promise</code>
Update a dictionary item value for a particular service and version.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Fulfil**: [<code>Response</code>](#Response)  
**See**: https://docs.fastly.com/api/config#dictionary_item_34c884a7cdce84dfcfd38dac7a0b5bb0  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The current version of a service. |
| name | <code>string</code> | The name of the dictionary. |
| key | <code>string</code> | The key to update data under. |
| value | <code>string</code> | The value to update the dictionary with. |

**Example**  
```js
instance.updateDictItem(1, 'extensions', 'html', 'text/html')
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+deleteDictItem"></a>

#### fastly.deleteDictItem(version, name, key) ⇒ <code>Promise</code>
Delete a dictionary item for a particular service and version.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Fulfil**: [<code>Response</code>](#Response)  
**See**: https://docs.fastly.com/api/config#dictionary_item_664347e743b8eafc9a93c729d9da0427  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The current version of a service. |
| name | <code>string</code> | The name of the dictionary. |
| key | <code>string</code> | The key to update data under. |

**Example**  
```js
instance.deleteDictItem('34', 'extensions', 'html')
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+writeDictItem"></a>

#### fastly.writeDictItem(version, name, key, value) ⇒ <code>Promise</code>
Safely create, update or delete a dictionary item in a named dictionary.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>number</code> | Service version to use for dictionary lookup. |
| name | <code>string</code> | Name of the dictionary (not ID). |
| key | <code>string</code> | Key to create, update or delete. |
| value | <code>string</code> | Value to update. Empty strings will delete the dictionary entry. |

<a name="Fastly+readDictionaries"></a>

#### fastly.readDictionaries(version) ⇒ <code>Promise</code>
List all dictionaries for a particular service and version.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**See**: https://docs.fastly.com/api/config#dictionary_6d2cc293b994eb8c16d93e92e91f3915  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The current version of a service. |

**Example**  
```js
instance.readDictionaries('12')
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+readDictionary"></a>

#### fastly.readDictionary(version, name) ⇒ <code>Promise</code>
Get details of a single dictionary.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**See**: https://docs.fastly.com/api/config#dictionary_0e16df083830ed3b6c30b73dcef64014  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The current version of a service. |
| name | <code>string</code> | Name of the dictionary. |

**Example**  
```js
instance.readDictionary('12', 'extensions')
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+createDictionary"></a>

#### fastly.createDictionary(version, data) ⇒ <code>Promise</code>
Create a new dictionary for a particular service and version.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The reponse object.  
**Fulfil**: [<code>Response</code>](#Response)  
**See**: https://docs.fastly.com/api/config#dictionary_7d48b87bf82433162a3b209292722125  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>number</code> | The version number (current if omitted). |
| data | <code>Object</code> | The dictionary definition. |

<a name="Fastly+updateDictionary"></a>

#### fastly.updateDictionary(version, name, data) ⇒ <code>Promise</code>
Update a dictionary for a particular service and version.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Fulfil**: [<code>Response</code>](#Response)  
**See**: https://docs.fastly.com/api/config#dictionary_8c9da370b1591d99e5389143a5589a32  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The current version of a service. |
| name | <code>string</code> | The name of the dictionary. |
| data | <code>Object</code> | The data to be sent as the request body. |

**Example**  
```js
instance.updateDictionary('34', 'old-name', { name: 'new-name' })
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+deleteDictionary"></a>

#### fastly.deleteDictionary(version, name) ⇒ <code>Promise</code>
Delete a dictionary for a particular service and version.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Fulfil**: [<code>Response</code>](#Response)  
**See**: https://docs.fastly.com/api/config#dictionary_8c9da370b1591d99e5389143a5589a32  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The current version of a service. |
| name | <code>string</code> | The name of the dictionary. |

**Example**  
```js
instance.deleteDictionary('34', 'extensions')
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+readBackends"></a>

#### fastly.readBackends(version) ⇒ <code>Promise</code>
List all backends for a particular service and version.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**See**: https://docs.fastly.com/api/config#backend_fb0e875c9a7669f071cbf89ca32c7f69  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The current version of a service. |

**Example**  
```js
instance.readBackends('12')
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+updateBackend"></a>

#### fastly.updateBackend(version, name, data) ⇒ <code>Promise</code>
Update the backend for a particular service and version.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Fulfil**: [<code>Response</code>](#Response)  
**See**: https://docs.fastly.com/api/config#backend_fb3b3529417c70f57458644f7aec652e  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The current version of a service. |
| name | <code>string</code> | The name of the backend. |
| data | <code>Object</code> | The data to be sent as the request body. |

**Example**  
```js
instance.updateBackend('34', 'slow-server', { name: 'fast-server' })
   .then(res => {
     console.log(res.data);
   })
   .catch(err => {
     console.log(err.message);
   });
```
<a name="Fastly+createBackend"></a>

#### fastly.createBackend(version, data) ⇒ <code>Promise</code>
Create a new backend for a particular service and version.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The reponse object.  
**Fulfil**: [<code>Response</code>](#Response)  
**See**: https://docs.fastly.com/api/config#backend_85c170418ee71191dbb3b5046aeb6c2c  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>number</code> | The version number (current if omitted). |
| data | <code>Object</code> | The backend definition. |

<a name="Fastly+createSnippet"></a>

#### fastly.createSnippet(version, data) ⇒ <code>Promise</code>
Create a snippet for a particular service and version.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**See**: https://docs.fastly.com/api/config#snippet_41e0e11c662d4d56adada215e707f30d  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The current version of a service. |
| data | [<code>Snippet</code>](#Snippet) | The data to be sent as the request body. |

**Example**  
```js
instance.createSnippet('36', {
    name: 'your_snippet',
    priority: 10,
    dynamic: 1,
    content: 'table referer_blacklist {}',
    type: 'init'
  })
  .then(res => {
    console.log(res.data);
  })
  .catch(err => {
    console.log(err.message);
  });
```
<a name="Fastly+updateSnippet"></a>

#### fastly.updateSnippet(version, name, data) ⇒ <code>Promise</code>
Update a VCL snippet for a particular service and version.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Fulfil**: [<code>Response</code>](#Response)  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The current version of a service. |
| name | <code>string</code> | The name of the snippet to update. |
| data | [<code>Snippet</code>](#Snippet) | The data to be sent as the request body. |

<a name="Fastly+createVCL"></a>

#### fastly.createVCL(version, data) ⇒ <code>Promise</code>
Create custom VCL for a particular service and version.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Fulfil**: [<code>Response</code>](#Response)  
**See**: https://docs.fastly.com/api/config#vcl_7ade6ab5926b903b6acf3335a85060cc  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The current version of a service. |
| data | [<code>VCL</code>](#VCL) | The data to be sent as the request body. |

<a name="Fastly+updateVCL"></a>

#### fastly.updateVCL(version, name, data) ⇒ <code>Promise</code>
Update custom VCL for a particular service and version.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Fulfil**: [<code>Response</code>](#Response)  
**See**: https://docs.fastly.com/api/config#vcl_0971365908e17086751c5ef2a8053087  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The current version of a service. |
| name | <code>string</code> | The name of the VCL to update. |
| data | [<code>VCL</code>](#VCL) | The data to be sent as the request body. |

<a name="Fastly+setMainVCL"></a>

#### fastly.setMainVCL(version, name) ⇒ <code>Promise</code>
Define a custom VCL to be the main VCL for a particular service and version.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Fulfil**: [<code>Response</code>](#Response)  
**See**: https://docs.fastly.com/api/config#vcl_5576c38e7652f5a7261bfcad41c0faf1  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The current version of a service. |
| name | <code>string</code> | The name of the VCL to declare main. |

<a name="Fastly+transact"></a>

#### fastly.transact(operations, activate) ⇒ <code>Object</code>
Creates a new version, runs the function `operations` and then
optionally activates the newly created version. This function
is useful for making modifications to a service config.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Object</code> - The return value of the wrapped function.  

| Param | Type | Default | Description |
| --- | --- | --- | --- |
| operations | <code>function</code> |  | A function that performs changes on the service config. |
| activate | <code>boolean</code> | <code>true</code> | Set to false to prevent automatic activation. |

**Example**  
```javascript
await fastly.transact(async (newversion) => {
 await fastly.doSomething(newversion);
});
// new version has been activated
```
<a name="Fastly+dryrun"></a>

#### fastly.dryrun(operations) ⇒ <code>Object</code>
See `transact`, but this version does not activate the created version.

**Kind**: instance method of [<code>Fastly</code>](#Fastly)  
**Returns**: <code>Object</code> - Whatever `operations` returns.  
**See**: #transact  

| Param | Type | Description |
| --- | --- | --- |
| operations | <code>function</code> | The operations that should be applied to the cloned service config version. |

<a name="CreateFunction"></a>

### CreateFunction ⇒ <code>Promise</code>
A function that creates a resource of a specific type. If a resource of that
name already exists, it will reject the returned promise with an error.

**Kind**: global typedef  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Fulfil**: [<code>Response</code>](#Response)  
**Reject**: [<code>FastlyError</code>](#FastlyError)  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The service config version to operate on. Needs to be checked out. |
| data | <code>Object</code> | The data object describing the resource to be created |
| data.name | <code>string</code> | The name of the resource to be created |

<a name="UpdateFunction"></a>

### UpdateFunction ⇒ <code>Promise</code>
A function that updates an already existing resource of a specific type.
If no resource of that name exists, it will reject the returned promise with an error.

**Kind**: global typedef  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Fulfil**: [<code>Response</code>](#Response)  
**Reject**: [<code>FastlyError</code>](#FastlyError)  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The service config version to operate on. Needs to be checked out. |
| name | <code>string</code> | The name of the resource to be updated. The old name in case of renaming something. |
| data | <code>Object</code> | The data object describing the resource to be updated |
| data.name | <code>string</code> | The new name of the resource to be updated |

<a name="ReadFunction"></a>

### ReadFunction ⇒ <code>Promise</code>
A function that retrieves a representation of a resource of a specific type.
If no resource of that name exists, it will reject the returned promise with an error.

**Kind**: global typedef  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Fulfil**: [<code>Response</code>](#Response)  
**Reject**: [<code>FastlyError</code>](#FastlyError)  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The service config version to operate on. Needs to be checked out. |
| name | <code>string</code> | The name of the resource to be retrieved. |

<a name="ListFunction"></a>

### ListFunction ⇒ <code>Promise</code>
A function that retrieves a list of resources of a specific type.

**Kind**: global typedef  
**Returns**: <code>Promise</code> - The response object representing the completion or failure.  
**Reject**: [<code>FastlyError</code>](#FastlyError)  

| Param | Type | Description |
| --- | --- | --- |
| version | <code>string</code> | The service config version to operate on. Needs to be checked out. |

<a name="FastlyError"></a>

### FastlyError : <code>Object</code>
The FastlyError class describes the most common errors that can occur
when working with the Fastly API. Using `error.status`, the underlying
HTTP status code can be retrieved. Known error status codes include:
- 400: attempting to activate invalid VCL
- 401: invalid credentials
- 404: resource not found
- 409: confict when trying to POST a resource that already exists
- 422: attempting to modify a service config that is not checked out
- 429: rate limit exceeded, try again later

**Kind**: global typedef  
**Properties**

| Name | Type | Description |
| --- | --- | --- |
| status | <code>Number</code> | The HTTP status code from the server response, e.g. 200. |
| data | <code>Object</code> | the parsed body of the HTTP response |
| code | <code>string</code> | a short error message |
| message | <code>string</code> | a more detailed error message |

<a name="Response"></a>

### Response : <code>Object</code>
**Kind**: global typedef  
**Properties**

| Name | Type | Description |
| --- | --- | --- |
| status | <code>Number</code> | The HTTP status code from the server response, e.g. 200. |
| statusText | <code>string</code> | The HTTP status text, e.g. 'OK' |
| headers | <code>Object</code> | The HTTP headers of the reponse |
| config | <code>Object</code> | The original request configuration used for the HTTP client |
| request | <code>Object</code> | the HTTP request |
| data | <code>Object</code> | the parsed body of the HTTP response |

<a name="Versions"></a>

### Versions : <code>Object</code>
Describes the most relevant versions of the service.

**Kind**: global typedef  
**Properties**

| Name | Type | Description |
| --- | --- | --- |
| latest | <code>number</code> | the latest version of the service |
| active | <code>number</code> | the currently active version number |
| current | <code>number</code> | the latest editable version number |

<a name="DictUpdate"></a>

### DictUpdate : <code>Object</code>
**Kind**: global typedef  
**Properties**

| Name | Type | Description |
| --- | --- | --- |
| op | <code>String</code> | The operation: `create`, `update`, or `delete` |
| item_key | <code>String</code> | the lookup key |
| item_value | <code>String</code> | the dictionary value |

<a name="Snippet"></a>

### Snippet : <code>Object</code>
**Kind**: global typedef  
**Properties**

| Name | Type | Description |
| --- | --- | --- |
| name | <code>String</code> | The name of the snippet, as visible in the Fastly UI |
| content | <code>String</code> | The VCL body of the snippet |

<a name="VCL"></a>

### VCL : <code>Object</code>
**Kind**: global typedef  
**Properties**

| Name | Type | Description |
| --- | --- | --- |
| name | <code>String</code> | The name of the VCL, as visible in the Fastly UI. Note: setting the name to 'main' here won't make it the main VCL, unless you also call `setMainVCL`. |
| content | <code>String</code> | The VCL body of the custom VCL |


## Tests

To run the test suite, first install the dependencies, then run the [`npm test` command](https://docs.npmjs.com/cli/test):

```bash
$ npm install
$ npm test
```

## Contribute

PRs accepted. I am open to suggestions in improving this library. Commit by:

```bash
$ npm run commit
```

## License

Licensed under the [MIT License](LICENSE) © 2017 Philipp Schulte
