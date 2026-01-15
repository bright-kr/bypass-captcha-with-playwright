# Playwright로 CAPTCHA 우회하기

[![Promo](https://github.com/bright-kr/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.co.kr/)

이 가이드는 Playwright를 사용해 CAPTCHA를 우회하고 Web스크레이핑 작업이 중단 없이 원활하게 실행되도록 보장하는 방법을 설명합니다:

- [CAPTCHA란 무엇이며 우회할 수 있습니까?](#what-are-captchas-and-can-you-bypass-them)
- [Playwright로 CAPTCHA 우회: 단계별 튜토리얼](#playwright-bypass-captcha-step-by-step-tutorial)
  - [Step #1: Node.js 프로젝트 초기화](#step-1-initialize-your-nodejs-project)
  - [Step #2: Playwright Extra 설치](#step-2-install-playwright-extra)
  - [Step #3: Playwright 스크립트 설정](#step-3-set-up-your-playwright-script)
  - [Step #4: 브라우저 자동화 로직 구현](#step-4-implement-the-browser-automation-logic)
  - [Step #5: Playwright Stealth 플러그인 설치](#step-5-install-the-playwright-stealth-plugin)
  - [Step #6: Stealth 설정 등록](#step-6-register-the-stealth-settings)
  - [Step #7: 봇 탐지 테스트 반복](#step-7-repeat-the-bot-detection-test)
- [Playwright CAPTCHA Solver 솔루션이 작동하지 않으면 어떻게 하나요?](#what-if-the-playwright-captcha-solver-solution-does-not-work)
 
## What Are CAPTCHAs and Can You Bypass Them?

[CAPTCHA](https://brightdata.co.kr/blog/web-data/what-is-a-captcha)는 “Completely Automated Public Turing tests to tell Computers and Humans Apart”의 약자로, 인간 사용자와 자동화된 봇을 구분하기 위해 사용되는 테스트입니다. 사람은 보통 쉽게 해결할 수 있지만, 기계는 해결하기 어렵도록 설계되었습니다.

![CAPTCHA 예시](https://github.com/bright-kr/bypass-captcha-with-playwright/blob/main/images/Example-of-a-CAPTCHA.png)

Google reCAPTCHA, hCaptcha, BotDetect는 가장 인기 있는 CAPTCHA 제공업체 중 일부입니다. 이들은 일반적으로 아래 CAPTCHA 유형 중 하나 이상을 지원합니다:

- **이미지 기반 챌린지:** 사용자는 이미지 그리드에서 특정 객체를 식별해야 합니다.
- **텍스트 기반 챌린지:** 사용자는 왜곡된 문자와 숫자 시퀀스를 입력해야 합니다.
- **오디오 기반 챌린지:** 사용자는 들리는 단어를 입력해야 합니다.
- **퍼즐 챌린지:** 사용자는 조각을 제자리에 슬라이드하는 것과 같은 간단한 퍼즐을 해결해야 합니다.

CAPTCHA는 폼 제출의 마지막 단계와 같이 특정 사용자 흐름의 일부일 수 있습니다:

![폼 제출 프로세스의 한 단계로서의 CAPTCHA 예시](https://github.com/bright-kr/bypass-captcha-with-playwright/blob/main/images/CAPTCHA-as-a-step-of-a-form-submission-process-example.png)

이러한 경우 CAPTCHA는 항상 표시되며 봇이 피할 수 없습니다. 그러나 CAPTCHA-solving 라이브러리와 소프트웨어를 통합하여 이를 자동화하거나, 사람 운영자가 실시간으로 이러한 챌린지를 해결하는 서비스에 의존할 수 있습니다.

CAPTCHA는 또한 [web application firewalls](https://www.cloudflare.com/learning/ddos/glossary/web-application-firewall-waf/)와 같은 더 광범위한 안티봇 솔루션의 일부로도 흔히 사용됩니다:

![Web Application Firewall 예시](https://github.com/bright-kr/bypass-captcha-with-playwright/blob/main/images/Example-of-a-Web-Application-Firewall-1024x488.png)

이러한 시스템은 사용자가 봇일 가능성이 있다고 의심할 때 동적으로 CAPTCHA를 표시합니다. 이런 상황에서는 봇이 인간 행동을 모방하도록 만들어 CAPTCHA를 우회할 수 있습니다.

## Playwright Bypass CAPTCHA: Step-By-Step Tutorial

CAPTCHA를 피하기 위한 효율적인 접근법은, 인간과 유사한 브라우ザフィンガープリント를 사용하면서 자동화 스크립트에서 인간 행동을 시뮬레이션하는 것입니다. [Playwright](https://playwright.dev/)는 선도적인 브라우저 자동화 라이브러리이며, 이 목적에 가장 적합한 도구 중 하나입니다.

이 가이드의 튜토리얼 섹션에서는 Playwright CAPTCHA 우회 로직을 구현하는 방법을 설명합니다.

### Step #1: Initialize Your Node.js Project

이미 Playwright Web스크레이핑 또는 테스트 스크립트가 있다면 이 단계는 건너뛰어도 됩니다. 그렇지 않다면, Playwright CAPTCHA solver 프로젝트용 폴더를 만들고 터미널에서 해당 폴더로 이동합니다:

```bash
mkdir playwright_demo

cd playwright_demo
```

그 안에서 새로운 Node.js 프로젝트를 초기화합니다:

```bash
npm init -y
```

선호하는 JavaScript IDE에서 프로젝트 폴더를 열고 새 `script.js` 파일을 추가합니다.

![IDE에서 새 script.js 파일 추가](https://github.com/bright-kr/bypass-captcha-with-playwright/blob/main/images/Adding-a-new-script.js-file-in-the-IDE.png)

`package.json`을 여는 것을 잊지 말고 `"type": "module"`을 추가하여 [프로젝트를 모듈로 표시](https://nodejs.org/api/packages.html#type)합니다.

### Step #2: Install Playwright Extra

Playwright의 플러그인 지원 부재는 커뮤니티에서 [Playwright Extra](https://github.com/berstend/puppeteer-extra/tree/master/packages/playwright-extra#readme)로 보완했습니다.

프로젝트의 dependencies에 `playwright`와 [`playwright-extra`](https://www.npmjs.com/package/playwright-extra)를 추가합니다:

```bash
npm i playwright playwright-extra
```

### Step #3: Set Up Your Playwright Script

이제 Playwright가 CAPTCHA 챌린지를 해결할 수 있도록 스크립트를 초기화할 시간입니다. `playwright-extra`에서 제어하려는 브라우저를 import하려면, `script.js`에 다음 줄을 추가합니다:

```js
import { chromium } from "playwright-extra"
```

Playwright API를 사용해 인간과 유사한 상호작용을 수행할 새 [async](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) 함수를 초기화합니다:

```js
(async () => {

    // set up the browser and launch it

    const browser = await chromium.launch()

    // open a new blank page

    const page = await browser.newPage()

    // browser automation logic...

    // close the browser and release its resources

    await browser.close()

})()
```

이는 새 Chromium 인스턴스를 실행하고 새 페이지를 연 다음 브라우저를 종료합니다.

### Step #4: Implement the Browser Automation Logic

대상 사이트는 [bot.sannysoft.com](https://bot.sannysoft.com/)이며, 브라우저에서 일부 테스트를 실행해 사용자가 인간인지 봇인지 판별하는 특수 웹 페이지입니다. 로컬 브라우저에서 이 페이지에 접속해 보면 모든 테스트가 통과되는 것을 확인할 수 있습니다.

[`goto()`](https://playwright.dev/docs/api/class-page#page-goto) 메서드를 사용해 대상 페이지에 연결합니다:

```js
await page.goto("https://bot.sannysoft.com/")
```

이제 anti-bot 테스트 결과를 확인하기 위해 페이지 전체의 스크린샷을 생성합니다:

```js
await page.screenshot("results.png")
```

모두 합치면 다음과 같은 `script.js` 파일이 됩니다:

```js
import { chromium } from "playwright-extra"

(async () => {

    // set up the browser and launch it

    const browser = await chromium.launch()

    // open a new blank page

    const page = await browser.newPage()

    // navigate to the target page

    await page.goto("https://bot.sannysoft.com/")

    // take a screenshot of the entire page

    await page.screenshot({

        path: "results.png",

        fullPage: true

    })

    // close the browser and release its resources

    await browser.close()

})()
```

아래 명령으로 위 코드를 실행합니다:

```bash
node script.js
```

스크립트는 headless 모드에서 Chromium 인스턴스를 열고, 원하는 페이지에 방문해 스크린샷을 찍은 뒤 브라우저를 종료합니다. 스크립트 실행이 끝나면 프로젝트 루트 폴더에 나타나는 `results.png` 파일을 열어 보면 다음과 같은 화면이 표시됩니다:

![results.png 파일 예시](https://github.com/bright-kr/bypass-captcha-with-playwright/blob/main/images/results.png-file-example-196x1024.png)

headless 모드의 기본 Playwright는 여러 테스트를 통과하지 못합니다. 이를 해결하려면 Stealth 플러그인을 사용합니다.

### Step #5: Install the Playwright Stealth Plugin

[Playwright Stealth](https://github.com/berstend/puppeteer-extra/tree/39248f1f5deeb21b1e7eb6ae07b8ef73f1231ab9/packages/puppeteer-extra-plugin-stealth)는 봇 탐지를 방지하기 위한 `playwright-extra` 플러그인입니다. 이는 여러 구성을 오버라이드하여, Playwright로 제어되고 있음에도 브라우저 인스턴스가 자연스럽게 보이도록 만듭니다.

Stealth 플러그인은 원래 Puppeteer Extra용으로 개발되었지만, Playwright Extra에서도 동작합니다. npm으로 설치합니다:

```bash
npm i puppeteer-extra-plugin-stealth
```

다음으로, `script.js` 파일에서 아래 줄로 Stealth 플러그인을 import합니다:

```js
import StealthPlugin from "puppeteer-extra-plugin-stealth"
```

### Step #6: Register the Stealth Settings

Playwright CAPCHA 우회 로직을 구현하려면, `use()` 메서드를 통해 `playwright-extra`에 Stealth 플러그인을 등록하기만 하면 됩니다:

```js
chromium.use(StealthPlugin())
```

이제 Playwright가 제어하는 브라우저는 인간 사용자가 실제로 사용하는 현실 세계의 브라우저처럼 보이게 됩니다.

### Step #7: Repeat the Bot Detection Test

현재 `script.js` 파일은 다음과 같아야 합니다:

```js
import { chromium } from "playwright-extra"

import StealthPlugin from "puppeteer-extra-plugin-stealth"

(async () => {

    // register the Stealth plugin

    chromium.use(StealthPlugin())

    // set up the browser and launch it

    const browser = await chromium.launch()

    // open a new blank page

    const page = await browser.newPage()

    // navigate to the target page

    await page.goto("https://bot.sannysoft.com/")

    // take a screenshot of the entire page

    await page.screenshot({

        path: "results.png",

        fullPage: true

    })

    // close the browser and release its resources

    await browser.close()

})()
```

스크립트를 다시 실행합니다:

```bash
node script.js
```

`results.png`를 다시 열어 보면, 이제 모든 봇 탐지 테스트가 통과된 것을 확인할 수 있습니다:

![results.png 파일 두 번째 예시 - 봇 탐지 테스트 통과](https://github.com/bright-kr/bypass-captcha-with-playwright/blob/main/images/results.png-file-second-example-bot-detection-tests-passed-229x1024.png)

## What If the Playwright CAPTCHA Solver Solution Does Not Work?

안티봇 도구가 주목하는 대상은 브라우저 설정만이 아닙니다. IP 평판은 또 다른 핵심 요소이며, 무료 라이브러리는 이 부분에 도움이 되지 않습니다.

한 번의 클릭만 필요한 단순한 CAPTCHA의 경우 [`puppeteer-extra-plugin-recaptcha`](https://github.com/berstend/puppeteer-extra/tree/39248f1f5deeb21b1e7eb6ae07b8ef73f1231ab9/packages/puppeteer-extra-plugin-recaptcha) 플러그인을 사용할 수 있습니다. 하지만 Cloudflare처럼 더 복잡한 도구를 다루는 경우에는 더 강력한 솔루션이 필요합니다.

실제 Playwright CAPTCHA solver를 찾고 있다면 Bright Data [web scraping solutions](https://brightdata.co.kr/products/web-unlocker)을 사용해 보십시오. 이는 reCAPTCHA, hCaptcha, Cloudflare Turnstile, AWS WAF Captcha 등 다양한 챌린지를 자동으로 처리하는 전용 [CAPTCHA-solving feature](https://brightdata.co.kr/products/web-unlocker/captcha-solver)를 통해 뛰어난 언락 기능을 제공합니다.

지금 등록하고 오늘 무료 체험을 시작해 보십시오.