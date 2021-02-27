---
title: Logging into Google with Selenium
date: 2017-07-20
description: Why fix it if it wasn't broken by me?
---

**Update from 2020: this workaround seems to have been patched by Google and is unlikely to work anymore. It's there for archival purposes only.**

Recently one of my scraping scripts which included [signing in](https://accounts.google.com/ServiceLogin) to Google account has broken. I've struggled for a couple hours to adapt the code to the new login form. So far, I thought I tried everything I could but got strange results to say the least. I could not even pass through username entry stage. [This page](https://support.google.com/accounts/answer/7338427?co=GENIE.Platform%3DDesktop&hl=en)
inspired me for a simple solution: why fix it if it's not broken _by me_ in the first place?

> You might still see the old sign-in page in these cases:
> • You use an older version of a browser
> • You've turned off JavaScript

Turning JS off was not an option ([here's why](https://stackoverflow.com/a/27787494/4617642)). I decided to go with the first one. Surprisingly, it worked just as expected! Now in the code, we fake `User-Agent` to be one of the Chrome very old versions, 19. I got the exact string from [here](http://www.useragentstring.com/pages/useragentstring.php?name=Chrome). It gets the job done: Google sends us the old log in form.

This is the code used to achieve the task:

```python
    from selenium.webdriver.common.by import By
    from selenium.webdriver import PhantomJS
    from selenium.webdriver.common.desired_capabilities import (
        DesiredCapabilities)
    from selenium.webdriver.support import expected_conditions
    from selenium.webdriver.support.ui import WebDriverWait


    def google_sign_in(gmail_username, gmail_pass):
        CHROME_UAG = ('Mozilla/5.0 (Macintosh; Intel Mac OS X 10_6_8) '
                      'AppleWebKit/535.7 (KHTML, like Gecko) '
                      'Chrome/16.0.912.36 Safari/535.7')
        caps = DesiredCapabilities.PHANTOMJS
        caps["phantomjs.page.settings.userAgent"] = CHROME_UAG

        # firstly, create webdriver for scraping
        driver = PhantomJS(
            executable_path='bin/phantomjs',
            desired_capabilities=caps
        )

        # set size so we that we (and browser) can see everything
        driver.set_window_size(2000, 2000)

        # go to login form
        GOOGLE_SIGN_IN_URL = 'https://accounts.google.com/ServiceLogin'
        driver.get(GOOGLE_SIGN_IN_URL)

        # enter username
        username_field = driver.find_element_by_id('Email')
        username_field.send_keys(gmail_username)
        username_field.submit()

        # enter password (but we have to wait until animation is finished)
        password_field = WebDriverWait(driver, 10).until(
            expected_conditions.element_to_be_clickable((By.ID, "Passwd"))
        )

        password_field.send_keys(gmail_pass)
        password_field.submit()
        return driver

```
