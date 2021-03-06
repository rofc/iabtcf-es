[![NPM version](https://img.shields.io/npm/v/@iabtcf/cmpapi.svg?style=flat-square)](https://www.npmjs.com/package/@iabtcf/cmpapi)
[![npm module downloads per month](http://img.shields.io/npm/dm/@iabtcf/cmpapi.svg?style=flat)](https://www.npmjs.org/package/@iabtcf/cmpapi)
[![Build](https://travis-ci.org/chrispaterson/iabtcf.svg?branch=master)](https://travis-ci.org/chrispaterson/iabtcf)
[![Coverage Status](https://coveralls.io/repos/github/chrispaterson/iabtcf/badge.svg?branch=master)](https://coveralls.io/github/chrispaterson/iabtcf?branch=master)

# @iabtcf/cmpapi

Ensures other in-page digital marketing technologies have access to CMP transparency and consent information for the [IAB's Transparency and Consent Framework (TCF)](https://github.com/InteractiveAdvertisingBureau/GDPR-Transparency-and-Consent-Framework).

# CmpApi

`CmpApi` is the only class needed to provide in-page digital marketing technologies access to a CMP transparency and consent information.
The process involves setting the state of a few properties and/or a validly ecnoded [TC string](https://github.com/InteractiveAdvertisingBureau/GDPR-Transparency-and-Consent-Framework/blob/master/TCFv2/IAB%20Tech%20Lab%20-%20Consent%20string%20and%20vendor%20list%20formats%20v2.md#about-the-transparency--consent-string-tc-string)

## Installation

npm
```
npm install @iabtcf/cmpapi --save
```

yarn
```
yarn add @iabtcf/cmpapi
```



## Create CmpApi

````javascript
import {CmpApi} from '@iabtcf/cmpapi';

/**
 * To create an instance of the CmpApi. Pass in your Cmp ID and the Cmp Version
 * to constructor. Custom commands are optional.
 */
const cmpApi = new CmpApi(1, 3);
````

During construction of the `CmpApi`, the `window.__tcfapi` stub is replaced with `CmpApi`'s own function
for handling `window.__tcfapi` command requests. Commands that were waiting to be executed in the stub are
filtered out if not valid. Ping and custom commands are executed and removed from the queue while
all other commands remain queued until a valid [TC string](https://github.com/InteractiveAdvertisingBureau/GDPR-Transparency-and-Consent-Framework/blob/master/TCFv2/IAB%20Tech%20Lab%20-%20Consent%20string%20and%20vendor%20list%20formats%20v2.md#about-the-transparency--consent-string-tc-string) is set.

**Note:** After creation, `window.__tcfapi` will respond to "ping" commands and custom commands only. All other commands
will be queue until `update()` is called for the first time.

## Set TC string
Create a **valid** [TC string](https://www.iabtcf.com/api/core/classes/tcmodel.html) and set it in `CmpApi`.

````javascript
cmpApi.update(encodedTCString);
````

## Show UI and Update TC string
`CmpApi` needs to know when you are going to show the user the CMP UI. The second parameter is a `boolean` letting `CmpApi` know that the UI is now visible to the user (it defaults to `false`).

````javascript
cmpApi.update(encodedTCString, true);
````

## GDPR doesn't apply
In the case that GDPR does not apply, simply update with null. That's all.

````javascript
cmpApi.update(null);
````

## Disabling the CmpApi
If, for any reason, we are unable to perform the operations in compliance with
the TCF and thus should not continue to serve page request commands (other than ping),
the `CmpApi` provides a disable method. Calling the disabled method will put the `CmpApi`
into a permanent error state. Only ping and custom commands will continue to be executed
for page requests.

````javascript
cmpApi.disable();
````

## Custom Commands
`CmpApi` has an optional parameter to pass in your map of custom commands.
CmpApi will not perform any validation on custom commands. The CMP is responsible for handling validations and errors. Custom function signatures
must have a callback and may define additonal params that will be passed from the calling script.

### Example
````javascript
import {CmpApi} from '@iabtcf/cmpapi';

const cmpApi = new CmpApi(1, 3, {

  'bingo': (callback, dogName) => {

    callback(`There was a farmer who had a dog, and ${dogName} was his name-o`);

  },

  'connectBones': (callback, startBone, endBone) => {

    callback(`The ${startBone} bone is connected to the ${endBone} bone.`);

  },

});

const songLyricCallback = (lyrics) => {

  if(success) {

    console.log(lyrics)

  } else {

    console.error('Error: could not get song lyrics')

  }

}

__tcfapi('bingo', 2, songLyricCallback, 'Bingo');
// ouput: There was a farmer who had a dog, and Bingo was his name-o

__tcfapi('connectBones', 2, songLyricCallback, 'knee', 'thigh');
// ouput: The knee bone is connected to the thigh bone
````

## CmpApi Examples

### Example 1: GDPR Applies and TC string exists and UI doesn't need to be shown

````javascript
import {CmpApi} from '@iabtcf/cmpapi';

// cmp ID 100, cmp version 2
const cmpApi = new CmpApi(100, 2);

/**
 * ... CMP makes determination not to show the UI
 *     based on the encodedTCString...
 */

// set string and the UI determination (false)
cmpApi.update(encodedTCString, false);

````

### Example 2: GDPR Applies and the UI needs to show

````javascript
import {CmpApi} from '@iabtcf/cmpapi';

// cmp ID 100, cmp version 2
const cmpApi = new CmpApi(100, 2);

/**
 * ... CMP makes determination to show the UI
 *     based on the encodedTCString...
 */

// set string and the UI determination (false)
cmpApi.update(encodedTCString, true);

/**
 * ... CMP waits for user to make their
 *     choices and encodes a new TC string...
 */

// update the cmpApi with the new value and in this case the
// second parameter doesn't matter
cmpApi.update(newEncodedTCString);

````

### Example 3: GDPR Does not apply to this user

````javascript
import {CmpApi} from '@iabtcf/cmpapi';

// cmp ID 100, cmp version 2
const cmpApi = new CmpApi(100, 2);

/**
 * ... CMP makes determination not that gdprApplies=false
 */

// Setting this to null sets gdprApplies=false
cmpApi.update(null);

````
