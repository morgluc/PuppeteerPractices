### Waiting for the page to finish rendering
[https://pptr.dev/api/puppeteer.page.goto/](https://pptr.dev/api/puppeteer.page.goto/)

The `page.goto()` method accepts an optional parameter which describes how long to wait before resolving the promise returned by `page.goto()`.
Use `networkidle0` to wait for all network traffic to finish, plus an additional half second (see docs). Using 'domcontentloaded' does not wait for your React page to finish rendering.

```
beforeAll(async () => {
	await page.goto('http://localhost:3000', {waitUntil: 'networkidle0'});
});
```

### Waiting for the effects of a button click
[https://pptr.dev/api/puppeteer.page.click/#remarks](https://pptr.dev/api/puppeteer.page.click/#remarks)

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

