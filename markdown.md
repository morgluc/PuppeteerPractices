Some tips about Puppeteer from Lucas Morgan (morgluc).

### Waiting for the page to finish rendering

[https://pptr.dev/api/puppeteer.page.goto/](https://pptr.dev/api/puppeteer.page.goto/)

The `page.goto()` method accepts an optional parameter which describes how long to wait before resolving the promise returned by `page.goto()`.
Use `networkidle0` to wait for all network traffic to finish, plus an additional half second (see docs). Using `domcontentloaded` does not wait for your React page to finish rendering, which means you have to write extra code to determine if the page is ready for measuring.

```
beforeAll(async () => {
	await page.goto('http://localhost:3000', {waitUntil: 'networkidle0'});
});
```

### Waiting for the effects of a button click

I see a lot of this strategy when asserting the effects of page interactions, where awaiting the interaction happens first, then the effect is measured.
```
// Assert
await page.click('.myButton');
await page.waitForSelector('.myConditionallyVisibleElement', {visible: true});
```

This causes a race between the effects of the button click and the `page.click()` promise resolving. The promise doesn't always win the race, so it's better to do this:
```
// Assert
await Promise.all([
	page.waitForSelector('.myConditionallyVisibleElement', {visible: true}),
	page.click('.myButton')
]);
```
In this way, the effect you're measuring doesn't race with the promise from `.click()`.   
[Puppeteer docs describe this, too.](https://pptr.dev/api/puppeteer.page.click/#remarks)

### Waiting for elements to appear or disappear

[https://pptr.dev/api/puppeteer.waitforselectoroptions/](https://pptr.dev/api/puppeteer.waitforselectoroptions/)  
`page.waitForSelector()` has some options to wait for elements to appear or disappear, but the options are confusing.


`page.waitForSelector('.myClass', {visible: true});` will tell Puppeteer to wait until the timeout (default 30 seconds) for `.myClass` to appear on the page.

Sometimes I see `page.waitForSelector('.myClass', {visible: false})` to wait for some element to disappear from the page, which seems intuitive but the option doesn't behave this way.  
To wait for an element to disappear use `page.waitForSelector({hidden: true});`
