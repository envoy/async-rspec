# Async::RSpec

Provides useful `RSpec.shared_context`s for testing code that builds on top of [async].

[async]: https://github.com/socketry/async

[![Build Status](https://secure.travis-ci.org/socketry/async-rspec.svg)](http://travis-ci.org/socketry/async-rspec)
[![Code Climate](https://codeclimate.com/github/socketry/async-rspec.svg)](https://codeclimate.com/github/socketry/async-rspec)
[![Coverage Status](https://coveralls.io/repos/socketry/async-rspec/badge.svg)](https://coveralls.io/r/socketry/async-rspec)

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'async-rspec'
```

And then execute:

	$ bundle

Or install it yourself as:

	$ gem install async-rspec

## Usage

### Leaks

Leaking sockets and other kinds of IOs is a problem for long running services. `Async::RSpec::Leaks` tracks all open sockets both before and after the spec. If any are left open, the spec fails.

```ruby
RSpec.describe "leaky ios" do
	include_context Async::RSpec::Leaks
	
	# The following fails:
	it "leaks io" do
		@input, @output = IO.pipe
	end
end
```

### Reactor

Many specs need to run within a reactor. A shared context is provided which includes all the relevant bits, including the above leaks checks.

```ruby
RSpec.describe IO do
	include_context Async::RSpec::Reactor
	
	let(:pipe) {IO.pipe}
	let(:input) {pipe.last}
	let(:output) {pipe.first}
	
	it "should send and receive data within the same reactor" do
		message = nil
		
		output_task = reactor.with(output) do |wrapper|
			message = wrapper.read(1024)
		end
		
		reactor.with(input) do |wrapper|
			wrapper.write("Hello World")
		end
		
		output_task.wait
		expect(message).to be == "Hello World"
end
```

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## License

Released under the MIT license.

Copyright, 2017, by [Samuel G. D. Williams](http://www.codeotaku.com/samuel-williams).

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
