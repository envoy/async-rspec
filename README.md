# Async::RSpec

Provides useful `RSpec.shared_context`s for testing code that builds on top of [async](https://github.com/socketry/async).

[![Development Status](https://github.com/socketry/async-rspec/workflows/Development/badge.svg)](https://github.com/socketry/async-rspec/actions?workflow=Development)

## Installation

``` shell
$ bundle add async-rspec
```

Then add this require statement to the top of `spec/spec_helper.rb`

``` ruby
require 'async/rspec'
```

## Usage

### Async Reactor

Many specs need to run within a reactor. A shared context is provided which includes all the relevant bits, including the above leaks checks. If your spec fails to run in less than 10 seconds, an `Async::TimeoutError` raises to prevent your test suite from hanging.

``` ruby
require 'async/io'

RSpec.describe Async::IO do
	include_context Async::RSpec::Reactor
	
	let(:pipe) {IO.pipe}
	let(:input) {Async::IO::Generic.new(pipe.first)}
	let(:output) {Async::IO::Generic.new(pipe.last)}
	
	it "should send and receive data within the same reactor" do
		message = nil
		
		output_task = reactor.async do
			message = input.read(1024)
		end
		
		reactor.async do
			output.write("Hello World")
		end
		
		output_task.wait
		expect(message).to be == "Hello World"
		
		input.close
		output.close
	end
end
```

### Changing Timeout

You can change the timeout by specifying it as an option:

``` ruby
RSpec.describe MySlowThing, timeout: 60 do
	# ...
end
```

### File Descriptor Leaks

Leaking sockets and other kinds of IOs are a problem for long running services. `Async::RSpec::Leaks` tracks all open sockets both before and after the spec. If any are left open, a `RuntimeError` is raised and the spec fails.

``` ruby
RSpec.describe "leaky ios" do
	include_context Async::RSpec::Leaks
	
	# The following fails:
	it "leaks io" do
		@input, @output = IO.pipe
	end
end
```

In some cases, the Ruby garbage collector will close IOs. In the above case, it's possible that just writing `IO.pipe` will not leak as Ruby will garbage collect the resulting IOs immediately. It's still incorrect to not close IOs, so don't depend on this behaviour.

### Allocations

This functionality was moved to [`rspec-memory`](https://github.com/socketry/rspec-memory).

## Contributing

1.  Fork it
2.  Create your feature branch (`git checkout -b my-new-feature`)
3.  Commit your changes (`git commit -am 'Add some feature'`)
4.  Push to the branch (`git push origin my-new-feature`)
5.  Create new Pull Request

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
