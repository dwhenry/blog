# Clear as mud

I have found the most worry thing to find in a codebase is someone being clever. This can take many forms, some of the most common being:

* the overuse of metaprogramming
* multiple interdependent classes
* reliance on special case behaviour of core ruby libraries that few people know about

I am really talking about anything that makes it harder for the next programmer to understand to understand what your intention was when you wrote the code.

I have recently been working my way through the (sidekiq)[https://github.com/mperham/sidekiq] codebase, and as a whole have found it to be well written and easy (relatively for the complexity of the problem) to understand. It does contain segments of code that I feel don't add value but instead hide intent.

```
def actions
  [action1, action2]
end

def invoke(*args)
  chain = actions.dup
  traverse_chain = lambda do
    if chain.empty?
      yield
    else
      chain.shift.call(*args, &traverse_chain)
    end
  end
  traverse_chain.call
end
```

Don't understand what the above code is doing? It took me quite a while to understand what the intent is of this code. It is used to created nested calls from an array of actions all of which wrap the initial block that was passed in. so if I was to expand this out based on the code above it would look like:

```
def invoke(*args)
  action1.call(*args) do
    action2.call(*args) do
      yield
    end
  end
end
```

The code style above is taken from functional programming and if you showed it to a functional programmer they would most likely recognise the pattern straight away. However, a majority of people who see this code will be ruby developers.

So can we refactor this code to achieve the same outcome and ensuring the next programmer can easily understand the intent?

```
def invoke(*args, &block)
  CallChain.new(actions, *args, &block).call
end

class CallChain
  def initialize(chain, *args, &block)
    @chain = chain.dup
    @args = args
    @block = block
  end

  def call
    _callable
  end

  def _callable
    if @chain.empty?
      @block.call
    else
      @chain.shift.call(*@args, &method(:_callable))
    end
  end
end
```

It's still complicated code it's solving a difficult problem, but I think the extra clarity that comes from expanding this code out is worth it.  What are your thought?
