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
…/frontend/dominoes-dr-market master  ? ❯ npm run web

> dominoes-dr-market@1.0.0 web
> expo start --web

Starting project at /home/ysl/repos/dr-market/frontend/dominoes-dr-market
Starting Metro Bundler
▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
█ ▄▄▄▄▄ █▄▄▄ ▀█▄█▄█ ▄▄▄▄▄ █
█ █   █ ██▄▀ █ ▀▄▄█ █   █ █
█ █▄▄▄█ ██▀▄ ▄███▀█ █▄▄▄█ █
█▄▄▄▄▄▄▄█ ▀▄█ ▀▄█▄█▄▄▄▄▄▄▄█
█  █▄ ▀▄▀█▄▀█▄▀█ ▀█▄█▀█▀▀▄█
█▀▀▄▀▀▄▄  ▄██▄█▄▄ ▀███▄▀▀ █
█▀██▀▀█▄▄▄▄ █ ▀▄ █ ▄▀▀█▀ ██
█ ▄▀▄▄▄▄▄█▄ █▀██ ▄▀ ██▄▀  █
█▄█▄███▄█▀ █▄▀  █ ▄▄▄  ▄▀▄█
█ ▄▄▄▄▄ ███▀▄ ▀ █ █▄█ ███▄█
█ █   █ █ ▄█ ▀▀█▄ ▄  ▄ █▀▀█
█ █▄▄▄█ █▀▀█  █▄ ▄█▀▀▄█   █
█▄▄▄▄▄▄▄█▄▄█▄██▄▄▄▄█▄▄███▄█

› Metro waiting on exp://192.168.100.183:8081
› Scan the QR code above with Expo Go (Android) or the Camera app (iOS)

› Web is waiting on http://localhost:8081

› Using Expo Go
› Press s │ switch to development build

› Press a │ open Android
› Press w │ open web

› Press j │ open debugger
› Press r │ reload app
› Press m │ toggle menu
› shift+m │ more tools
› Press o │ open project code in your editor

› Press ? │ show all commands

Logs for your project will appear below. Press Ctrl+C to exit.
λ Bundled 8885ms node_modules/expo-router/node/render.js (1049 modules)
Web node_modules/expo-router/entry.js ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓░ 96.5% (1020/1051)
Web Bundled 10295ms node_modules/expo-router/entry.js (1111 modules)
Web Bundled 4360ms node_modules/expo-router/entry.js (1112 modules)
 LOG  [web] Logs will appear in the browser console
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
