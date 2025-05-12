+++
title = 'on Dev Mode, A/B tests and Random Oracles'
date = 2025-05-10T10:34:04+01:00
draft = false
tags = [
  "DevMode",
  "ABTests",
  "RandomOracles",
  "continuousDelivery",
  "TypeScript",
  "figma",
  "hashing",
]
+++

In the past days I have been working on this new feature that requires some code changes around the code base. For the sake of simplicity of the article, I would call this feature *Dev Mode* as it is a concept that everybody recognizes, even before __*Figma®*__  existed. BUT I don't want their lawyers to find this article and [come to me with the whole cease and desist drama](https://techcrunch.com/2025/04/15/figma-sent-a-cease-and-desist-letter-to-lovable-over-the-term-dev-mode/) (can you even threaten an article? I hope not). Therefore, to avoid any *misunderstanding* with Figma, let's call this feature *__Fig Mode__* instead.

While developing *Fig Mode*, I used a feature flag to hide this feature from <del>Figma</del> the users, so we can release other stuff every day while our new feature is not production ready. 

The logic here is simple - You have your collection of Features, and the function `isEnabled` checks if the environment variable for that feature is set:

```ts
enum Feature {
 FIG_MODE = 'FIG_MODE',
 // … other features
}


function isEnabled(feature: Feature): boolean {
 return process.env[`FEATURE_FLAG_${feature}`] === 'true';
}


if (isEnabled(Feature.FIG_MODE)) {
 // custom logic for the feature
}

```

The thing is, we want to roll out *Fig Mode* to only a fraction of our users for now. The reasons to do this *A/B test* can be many:
- Does the feature improve or degrade UX?
- Has it a hidden bug that we can catch before it affects all users?
- How does this feature affect the workload in our systems?

To answer those questions we not only have to split our users properly, but also collect and analyse the metrics of our experiment - and that is a whole other topic that we could touch on in the future. For now, let's focus on the user splitting.

### Enabling *Fig Mode* to 10% of the users

At first, we could think of the following naive implementation: 

```ts
enum Feature {
 FIG_MODE = 'FIG_MODE',
 // ...
}


function abTest(percentage: number): boolean {
 return Math.random() * 100 < percentage;
}


function isEnabled(feature: Feature): boolean {
 if (feature === Feature.FIG_MODE) {
   return abTest(10); // true for ~10% of the calls
 }
 return process.env[`FEATURE_FLAG_${feature}`] === 'true';
}


if (isEnabled(Feature.FIG_MODE)) {
 // custom logic for the feature
}
```

1. We have our Feature Flags collection, represented here by the enum.
2. The function `abTest` is pretty simple, calling it has X% chance of returning `true`, based on a random number that goes from 0 to 100.
3. `isEnabled` now returns the abTest result if the feature is *Fig Mode*.

This implementation can look enough at a first sight, but more attentive eyes may have already identified that we have a problem. We have to guarantee that the user will always receive the same result. Visiting a website and using a feature, then 2 minutes later the feature is not there anymore, then 10 minutes later it's there again… It's not only bad UX but *can also cause inconsistency*. In other words, we don't want users to be *jumping from an experiment bucket to another*.

To avoid that, you could store the feature flag result in a *key/value storage* such as *__Redis__* (it's a good practice to have Feature Flags be short-lived anyway). So a key could be `features/{feature_name}/{user_id}` and it would store the boolean result of the first time you called `isEnabled` for this user.

But there is another way to accomplish that, with *__O(1)__* in space and no network calls to a distributed cache. You can rely on a *__Random Oracle__* to give you a deterministic (pseudo) random number derived from your user id. So this *‘magic’* function would work like this:

```ts
// returns a random number that is the same for the same input
function randomOracle(input: string): number;

randomOracle('someUserId'); // 83459
randomOracle('someUserId'); // 83459
randomOracle('someUserId'); // 83459
randomOracle('someOtherId'); // 26760
randomOracle('someOtherId'); // 26760
```

### Implementing a Random Oracle

How does one transforms a predictable input into an unpredictable output? *__Hashing!__* A cryptographic hash function will take an input and map it to a *random-looking value*, then we can digest it as hexadecimal, and from that we parse it to a number.

#### A Nodejs (Typescript) implementation would look like this:

```ts
import { createHash } from 'node:crypto';


const randomOracle = (input: string): number => {
 const hash = createHash('sha1').update(input).digest('hex');
 const hash52Bits = hash.slice(hash.length - 13);
 const randInt = parseInt('0x' + hash52Bits, 16);
 return randInt;
};
``` 

Some details about this implementation:
- __Why pick *sha1* as the hash function? Isn't it insecure?__
  - Honestly, you could pick any other one such as *sha256*, *sha512* or *MD5* as their output has enough entropy for our use case. Regarding security, *sha1* is indeed weak for security applications, but it's not the use case here. I picked *sha1* because its Node implementation is faster than the above mentioned.
- __Why specifically a 52-bits integer?__
  - A *sha1* hash has 40 hex digits (160 bits), so the biggest safe integer we can get, the better for the entropy, right? *Right.* However, the max safe integer you can represent in Javascript is 2^53 – 1, just 53 bits. So, we use 13 hex digits (52 bits) to stay within JavaScript's safe integer range, because 14 hex digits would go up to 56 bits, beyond the safe range.


### Full implementation

```ts
import { createHash } from 'node:crypto';

enum Feature {
 FIG_MODE = 'FIG_MODE',
 // ...
}

const randomOracle = (input: string): number => {
 const hash = createHash('sha1').update(input).digest('hex');
 const hash52Bits = hash.slice(hash.length - 13);
 const randInt = parseInt('0x' + hash52Bits, 16);
 return randInt;
};

function abTest(percentage: number, key: string): boolean {
 return randomOracle(key) % 100 < percentage;
}

function isEnabled(feature: Feature, userId: string): boolean {
 if (feature === Feature.FIG_MODE) {
   return abTest(10, `${feature}/${userId}`); // true for ~10% of the users
 }
 return process.env[`FEATURE_FLAG_${feature}`] === 'true';
}

if (isEnabled(Feature.FIG_MODE, userId)) {
 // custom logic for the feature
}
```

Note that we use `${feature}/${userId}` and not only the `userId` as the key for the random oracle, this is important because we want to have *different outputs for different flags*, so the key have to not only distinguish between users, *but also between experiments*.

Also note that the `abTest` function now uses the modulus operation (% 100) to reduce the int output to a value between 0 and 99, so we can compare it to a percentage threshold.

### Wrapping up

That’s the essence of applying Random Oracles in feature flags to have A/B tests. Using a *hash-based approach* lets us roll out features gradually without added infrastructure, 3rd party solutions, or inconsistent behavior. It’s simple and got the job done <del>for now<del>.
