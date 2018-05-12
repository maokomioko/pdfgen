# Pdfgen
A tool for creating PDFs using [Puppeteer](https://github.com/GoogleChrome/puppeteer) and headless Chrome.

# Dependencies
This gem requires that node be installed along with puppeteer.

### OSX (Homebrew)
To install node on OSX you can use Homebrew:
```bash
brew install nodejs
```

And then install puppeteer with npm:
```bash
npm install --save puppeteer
```

# Usage
Currently the only use case that is supported is rendering an HTML string. You can do that by using the
Pdfgen class:
```ruby
pdf_as_string = Pdfgen.new(html_to_turn_into_pdf).to_pdf
```
`to_pdf` returns the pdf as a string. This makes it easy to send to a client from a web server like
Rails or to save to a file if that is desired.

If you need to provide options such as page margins you can pass them as a hash to `to_pdf`. Any
options that work for [Puppeteer's `pdf` method](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagepdfoptions)
are accepted as they are passed through without modification.
```ruby
pdf_as_string = Pdfgen.new(html_to_turn_into_pdf).to_pdf(margins: { top: '1in', bottom: '1in' })
```

# Reference

All methods other than `to_pdf` are chainable. This allows for setting multiple configuration options before rendering the PDF.
It also does not matter the order that they are called in except that `to_pdf` must be last since it returns the PDF.

Example:
```ruby
Pdfgen.new(html).set_viewport(width: 1024, height: 768).wait_for_timeout(1000).emulate_media('screen').to_pdf(format: 'Letter')
```

## Chainable configuration methods

### debug_mode(debug_time)

Sets a couple of options so the browser window used to print the PDF can be seen and inspected. *debug_time* is
the amount of time in milliseconds to leave the browser window open before the PDF will be printed and the browser
closed. It also disables headless mode.

Example:
Turns on debug mode and waits for 5 minutes.
```ruby
Pdfgen.new(html).debug_mode(300_000).to_pdf
```

### emulate_media(media_type)

Configure the media type for the browser used to render the PDF. Media type can be `'screen'`, `'print'`, or `null`.
See [emulateMedia](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageemulatemediamediatype) for more info.

Example:
Set the media emulation type to print.
```ruby
Pdfgen.new(html).emulate_media('print').to_pdf
```

### launch_options(launch_options)

Configure the options used to launch the browser. Can be used to set extra args for the browser such as `--no-sandbox` or to
disable headless mode. *launch_options* is a hash that is passed directly to the [launch](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#puppeteerlaunchoptions)
Puppeteer method and as such takes the same arguments.

Example:
Adds '--no-sandbox' flag and turns off headless mode.
```ruby
Pdfgen.new(html).launch_options(args: ['--no-sandbox'], headless: false).to_pdf
```

### set_viewport(viewport_options)

Configure the viewport. *viewport_options* is a hash that is passed directly to the
[setViewport](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagesetviewportviewport)
Puppeteer method and as such takes the same arguments.

Example:
Sets the viewport width and height to 1024x768.
```ruby
Pdfgen.new(html).set_viewport(width: 1024, height: 768).to_pdf
```

### wait_for_timeout(wait_for_timeout)

Configure an optional timeout to wait for with the browser loaded before the PDF will be saved specified in
milliseconds. Used to allow the page to finish rendering if the default prints the PDF to fast.

Example:
Sets the wait timeout to 1.5 seconds.
```ruby
Pdfgen.new(html).wait_for_timeout(1500).to_pdf
```

## PDF rendering methods

### to_pdf(opts)

Configure the options for making the PDF. It allows for setting margins, a scale, specific page ranges, configuring
the paper type, and more. *opts* is a hash that is passed directly to the [pdf](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagepdfoptions)
Puppeteer method and as such takes the same arguments.

Example:
Configures the margins to be 1 inch all around.
```ruby
Pdfgen.new(html).to_pdf(margin: {top: '1in', right: '1in', bottom: '1in', left: '1in'})
```

# Future Development
In the future, allowing use of more features of Puppeteer is desired. These include taking a URL and
setting cookies. Instead of using a fixed make_pdf.js script, it will probably make sense to convert
to generating that javascript file on the fly using templates since additional options like allowing
the setting of cookies will require calling more functions than are currently being called.