+++
date = '2026-01-06T18:57:41-04:00'
draft = false
title = 'Pursuite of Frontend Happiness'
+++

I had abandoned all hope for frontend and maximized on minimalism by ditching all frameworks and returning to proverbial html, css and js.
This left me blindsided by the strides being made by the messy webdev and designers community while I went on learning how to write better code in the first place. I could also say that this gave time for technologies to rise and mature.
This week I came across React Native Expo and I decided that this was the Framework I was going to stick with and master in order to build cross-platform applications with ease.

## React Expo

I shouldn't need too many introductions right?
I went on npmjs.com directly and double checked every package name I was going to install, being back on npm-land feels dangerous after so many slipups and oopsies as of late.

```bash
npx create-expo --template tabs
```

Easy enough I was up and running... not without first backing out, deleting the tabs template and replacing with

```bash
npx rn-new@latest --nativewind
```

As I wanted to skip building my own css classes and token specifications, I decided to use NativeWind for styling.

```
…/frontend/dominoes-dr-market master  ? ❯ npm run web

> dominoes-dr-market@1.0.0 web
> expo start --web

Starting project at /home/ysl/repos/dr-market/frontend/dominoes-dr-market
Starting Metro Bundler
▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
█ ▄▄▄▄▄ █▄▄▄ ▀█▄█▄█ ▄▄▄▄▄ █
...

› Metro waiting on exp://192.168.100.183:8081
› Scan the QR code above with Expo Go (Android) or the Camera app (iOS)

› Web is waiting on http://localhost:8081
```

That output shows the link to the development workflow that they have with the app.

So they distribute this app called Expo Go on the client that serves as a sandbox for your development code to run on, as well as an interface to native APIs on the phone the app is installed on. This way you can test things like haptic feedback without having to compile, package and install a native app. Kind of brilliant, certainly innovative. I was immediately wondering "Did apple just let them do this?" but upon closer inspection you find that only a select reviewed libraries are allowed on  the platform by the Expo team, that's the quickest reasoning I can see, that or maybe apple has not caught up yet.
The point is, you can download that app for testing your mobile interface end to end.
Apparently its called Metro, searching more about it:
 <https://docs.expo.dev/guides/customizing-metro/>
In summary, It's a bundler that takes the js and css and bundles them into packages for each platform while doing a lot of magical React Expo stuff.

While searching I found this quote:

> What is Expo Go? What is a development build? My project opened in Expo Go, when do I need to switch to a development build?

Same questions I was asking myself. Have a read if you want to dive deeper into some of these details: <https://expo.dev/blog/expo-go-vs-development-builds>

## Development plan

Before the code:

- Document complete user journey from entry to exit
- Identify authentication gates
- Take inventory of your screens and paths
- Map conditional paths (authenticated vs. unauthenticated)
- Establish the data dependency of each page, Dto pattern, typescript interfaces, etc.
-

```plaintext
Public Routes:
├── / (landing - Next.js)
└── /about

Protected Routes (app.domain.com):
├── /dashboard
├── /[feature, eg. listings]/list
├── /[feature, eg. profiles]/[id]
├── /settings
│   ├── /settings/profile
│   └── /settings/security
└── /auth/callback
```

- Quantify: 5-15 distinct screens for MVP

### Tiered Components Design System

The whole time I couldn't help but ask, could some of these design patterns be useful elsewhere, without the React overhead?
And I'm referring to the terseness and the brevity of the code, the prop-components pattern, the encapsulated logic and behaviour of each component.

And I think I got it, I just had to extrapolate these qualities and write cshtml around it, which is what I was writing before.

Tiered Design System is based around priority classification:

Tier 1 (Build First):

- Button (primary, secondary, ghost variants)
- Input (text, email, password)
- Card container
- Layout wrapper
- Loading spinner

Tier 2 (Build Second):

- Modal/Dialog
- Dropdown/Select
- Table/List
- Form wrapper with validation

Tier 3 (Build Last):

- Toast notifications
- Tabs
- Accordion
- Complex data visualizations

Here is how I would define a css token spec if I had to:

```
export const tokens = {
  colors: {
    primary: { 50: '#...', 600: '#...', 900: '#...' },
    gray: { 50: '#...', 900: '#...' },
    error: '#...',
    success: '#...',
  },
  spacing: { xs: 4, sm: 8, md: 16, lg: 24, xl: 32 },
  radius: { sm: 4, md: 8, lg: 12 },
  typography: {
    h1: { size: 32, weight: 700, lineHeight: 1.2 },
    body: { size: 16, weight: 400, lineHeight: 1.5 },
  }
};
```

And I would probably have copilot generate the rest of that.

---

After that, we are mostly ready to implement the UI. Skip wireframing and all that, or use a tool like figma, your call. Total MVP timeline can be 20-30 hours of focused work.

## Okay, let's skip the UI coding

You don't want to see more of that. Let me give you my rundown of how I understood the system.

### First impressions: Layers and layers

Transpilers and abstractions, everywhere.

If you're using TypeScript (which you should be), you're hitting two transpilers on the JavaScript side alone:

1. TypeScript transpiler (TS → JS)
2. Babel (modern JS → compatible JS)

And that's *before* you even try to build and compile into native code.

React Native adds another layer: the Metro bundler takes your bundled JavaScript and packages it for iOS and Android, handling platform-specific transformations, code splitting, and all the other magic that lets you write once and run everywhere.

I mean, you are layers and layers removed from any actual system integration, but here, that is a positive. As a React Native Expo developer you want to make your screens and ship your product quick, not worry about the C code at the core of it.

### Configuration jungle

At first glance, the project root looks like a minefield of config files. Here's what each one actually does:

**app.json** - Expo's main configuration file. This is where you define your app name, version, icons, splash screens, and platform-specific settings. Think of it as the manifest for your entire app.

```json
{
  "expo": {
    "name": "your-app",
    "slug": "your-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "splash": {
      "image": "./assets/splash.png"
    },
    "platforms": ["ios", "android", "web"]
  }
}
```

**package.json** - Standard npm configuration. Dependencies, scripts, metadata. Nothing special here if you've worked with Node before.

**.env** - Environment variables for different deployment targets. API keys, endpoint URLs, feature flags. Keep this out of git.

**.babelrc** (or babel.config.js) - Babel configuration for JavaScript transpilation. Expo comes with a preset, but you can extend it with plugins. NativeWind, for example, adds its own Babel plugin here.

```json
{
  "presets": ["babel-preset-expo"],
  "plugins": ["nativewind/babel"]
}
```

### The router situation

Now we get to the interesting part: routing.

You've got **Expo Router** sitting on top, which itself uses **React Router** underneath. I mean, you are layers and layers removed from any actual system integration, but here, that is a positive. As a React Native Expo developer you want to make your screens and ship your product quick, not worry about the C code at the core of it.

#### Expo Router

Expo Router uses file-based routing, similar to Next.js. Your folder structure *is* your route structure:

```
app/
├── _layout.tsx          # Root layout
├── index.tsx            # Home screen (/)
├── about.tsx            # About screen (/about)
└── (tabs)/
    ├── _layout.tsx      # Tab layout
    ├── home.tsx         # /home
    └── profile.tsx      # /profile
```

The parentheses `(tabs)` create a route group without adding to the URL path. It's actually clever once you get used to it.

Navigation is declarative:

```tsx
import { Link } from 'expo-router';

<Link href="/about">About</Link>
```

Or programmatic:

```tsx
import { router } from 'expo-router';

router.push('/about');
```

#### React Router underneath

Expo Router is built on React Router v6, which means all the same concepts apply: routes, navigation, nested routes, URL parameters. But Expo Router abstracts most of it away with the file-based approach.

Under the hood, React Router handles:
- Route matching
- Navigation state management
- History management (important for web targets)
- Deep linking (crucial for mobile)

The abstraction is nice until you need to do something non-standard. Then you're debugging through layers of framework code trying to figure out which part is Expo and which part is React Router.

### React2Shell: When abstractions attack

Speaking of React Router, let's talk about CVE-2025-55182, informally known as **React2Shell**.

#### The problem

React2Shell is a CVSS 10.0 (maximum severity) pre-authentication RCE vulnerability affecting React Server Components. It was discovered by [Lachlan Davidson](https://github.com/lachlan2k/React2Shell-CVE-2025-55182-original-poc) and disclosed to the React team on November 29, 2025.

The core issue is an insecure deserialization vulnerability in React's "Flight" protocol, which handles server component communication.

#### How it was introduced

React Server Components (RSC) introduced a new way to render components on the server and stream them to the client. To make this work, React needed a protocol for serializing and deserializing component state between server and client.

The Flight protocol handles this serialization. But the deserialization logic had a critical flaw: it trusted the incoming data without proper validation.

An attacker could craft a malicious payload that, when deserialized, would execute arbitrary code on the server.

#### How it was found

Davidson found the vulnerability through security research focused on React's Server Components architecture. After responsible disclosure to Meta's Bug Bounty program on November 29, 2025, the vulnerability was publicly disclosed on December 3, 2025.

Within hours of public disclosure, security researchers published proof-of-concept exploits:
- [Moritz Sanft published working PoC code](https://github.com/moritz-sanft/react2shell-poc) on December 4
- [Davidson released his original PoC](https://github.com/lachlan2k/React2Shell-CVE-2025-55182-original-poc) on December 5

Several comprehensive technical writeups followed:
- [Wiz's deep-dive analysis](https://www.wiz.io/blog/nextjs-cve-2025-55182-react2shell-deep-dive) of the exploit mechanics
- [OX Security's granular technical breakdown](https://www.ox.security/blog/react2shell-going-granular-a-deep-deep-deep-technical-analysis-of-cve-2025-55182/)
- [Datadog Security Labs' analysis](https://securitylabs.datadoghq.com/articles/cve-2025-55182-react2shell-remote-code-execution-react-server-components/)
- [Trend Micro's PoC analysis](https://www.trendmicro.com/en_us/research/25/l/CVE-2025-55182-analysis-poc-itw.html)

#### Reproducing the vulnerability

The exploit works through several stages:

1. **Circular Reference Exploitation**: The attacker creates two multipart form chunks that reference each other:
   - Chunk 0 contains a malicious JSON payload
   - Chunk 1 points back to Chunk 0 using `$@0`

2. **Prototype Pollution**: By exploiting the `$1:__proto__:then` reference pattern, the attacker pollutes `Chunk.prototype.then`, setting a fake chunk's status to `'resolved_model'` to trigger `initializeModelChunk` with attacker-controlled data.

3. **Code Execution**: The crafted payload chains internal React gadgets to create a Promise-like object with an attacker-controlled `.then` property. During deserialization, React automatically resolves these Promise-like objects, which results in code execution.

Here's a simplified example of the vulnerable code path:

```javascript
// Vulnerable deserialization in React Flight
function parseModelString(response, parentObject, value) {
  if (value[0] === '$') {
    // Reference to another chunk
    const id = parseInt(value.substring(1), 16);
    const chunk = getChunk(response, id);

    // Unsafe: Automatically resolves Promise-like objects
    if (chunk.status === 'resolved_model') {
      return initializeModelChunk(chunk);  // RCE here
    }
  }
  return value;
}
```

The researchers were able to achieve RCE in lab environments by:

1. Setting up a vulnerable Next.js 15.x or React 19.x application
2. Sending a crafted multipart/form-data POST request to any Server Action endpoint
3. The payload triggers code execution before the Server Action is even validated

Default configurations were vulnerable. A standard Next.js app created with `create-next-app` and built for production could be exploited with zero code changes by the developer.

#### How it was patched

React released patches in versions 19.0.1, 19.1.2, and 19.2.1. The fix involved:

1. Strict validation of deserialized data structures
2. Removing automatic resolution of Promise-like objects during deserialization
3. Adding integrity checks for chunk references
4. Implementing allowlists for safe deserialization patterns

Next.js released corresponding patches in versions 15.1.4 and 16.0.1.

#### Real-world impact

The vulnerability didn't stay theoretical for long.

Within hours of disclosure on December 3, multiple China state-nexus threat groups began active exploitation, including Earth Lamia and Jackpot Panda, according to [AWS threat intelligence](https://aws.amazon.com/blogs/security/china-nexus-cyber-threat-groups-rapidly-exploit-react2shell-vulnerability-cve-2025-55182/).

[Google Threat Intelligence](https://cloud.google.com/blog/topics/threat-intelligence/threat-actors-exploit-react2shell-cve-2025-55182) identified campaigns deploying:
- MINOCAT tunneler
- SNOWLIGHT downloader
- HISONIC backdoor
- COMPOOD backdoor
- XMRIG cryptocurrency miners

Iran-nexus actors were also observed exploiting the vulnerability. [Trend Micro documented](https://www.trendmicro.com/en_us/research/25/l/CVE-2025-55182-analysis-poc-itw.html) campaigns executing Cobalt Strike beacons, deploying Nezha, Fast Reverse Proxy (FRP), and the Sliver payload.

CISA added it to the Known Exploited Vulnerabilities catalog, meaning federal agencies had to patch immediately.

### React Native implications

So what does this mean for React Native developers?

**Good news first**: React Native doesn't use React Server Components in the same way Next.js does. The Flight protocol that was vulnerable isn't part of standard React Native architecture.

**But** (there's always a but):

1. **Expo Router uses React Router** underneath, which shares some of the same deserialization patterns, though not the vulnerable Server Components code path.

2. **If you're using React 19.x** in your React Native project, you should still update to the patched versions. Even though the specific RCE vector doesn't apply, defense in depth matters.

3. **Framework coupling risk**: This vulnerability shows how deep the dependency chain goes. React Native → Expo Router → React Router → React core. A vulnerability in the core can have blast radius even if your specific use case doesn't trigger it.

4. **Ecosystem effects**: The React ecosystem moves fast. Security patches in React core often trigger cascading updates across the entire ecosystem. Expo, React Navigation, and other libraries all had to evaluate their exposure and release updates.

The practical impact for most React Native developers is minimal for this specific CVE, but it's a good reminder: keep your dependencies updated, understand what's in your dependency tree, and pay attention to security advisories.

## Closing thoughts

React Native Expo is a great framework for shipping cross-platform apps quickly. The abstraction layers are deep, but they're deep for a reason: they let you focus on building products instead of wrestling with platform-specific native code.

But those abstractions come with responsibility. You need to understand, at least roughly, what's happening under the hood. Not so you can optimize every render cycle, but so you can debug when things go wrong and assess security risks when vulnerabilities like React2Shell emerge.

The config files make sense once you understand their purpose. The router situation is actually elegant once you get past the initial learning curve. And the security landscape... well, that's just the reality of building on top of a massive, fast-moving ecosystem.

Keep your dependencies updated. Read the security advisories. Ship your product.

And maybe check twice before `npm install`-ing that random package.
