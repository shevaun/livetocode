---
layout: post
title: Testing Infinite Loops with RSpec
tags: rspec ruby
---

Recently I had to make a minor modification to a job scheduler; a ruby process that runs an infinite loop on startup, which polls the database for jobs that need processing every so often.

The code was easy to change but for a while I was stumped on how I could test my changes, since they were inside the loop.

```ruby
class Scheduling::Daemon
  def run
    # ... (code to set up polling period and calculate next tick)
    loop do
      if JobScheduler::PropertiesControl &&
	    java.lang.System.getProperties['jobScheduler.mode'] == 'stop'
		# ... (code to write to log etc)
        break
      end

      # ... (code to sleep for calculated period of time)
      JobScheduler::process_jobs
    end
  end
end
```

One option was to refactor the contents of the loop into a separate method and just test that, but I wanted to be able to test the entire run method, including the termination path. I also didn't want to make huge changes to this class as it's currently used in production by a lot of different systems and is a critical part of the system.

I realised that if I refactored the exit condition to be a method, I could stub it and get it to return false on its first invocation then true on its second invocation; ensuring the loop only gets executed once and then terminates.

Refactored class:

```ruby
class Scheduling::Daemon
  def run
    # ... (code to set up polling period and calculate next tick)
    loop do
      if daemon_received_stop_signal?
   		# ... (code to write to log etc)
        break
	  end

      # ... (code to sleep for calculated period of time)
      JobScheduler::process_jobs
    end
  end

  private

  def daemon_received_stop_signal?
    JobScheduler::PropertiesControl &&
	  java.lang.System.getProperties['jobScheduler.mode'] == 'stop'
  end
end
```

Now the spec is easy to write and run:

```ruby
describe Scheduling::Daemon do
  describe "#run" do
    before do
      Scheduling::Daemon.should_receive(:daemon_received_stop_signal?).
        and_return(false, true)  # execute loop once then exit
	end

    it "calls process_jobs" do
      JobScheduler.should_receive(:process_jobs)
      Scheduling::Daemon.run
    end
  end
end
```
