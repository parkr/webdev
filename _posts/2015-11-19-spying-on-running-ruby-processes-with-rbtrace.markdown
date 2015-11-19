---
layout: post
title: "Spying on running Ruby processes with rbtrace"
date: 2015-11-19 09:20:52 -0800
external-url: http://blog.honeybadger.io/spying-on-running-ruby-processes/
---

Memorable code:

```ruby
rbtrace -p 12345 -f
*** attached to process 12345
Fixnum#** <0.000010>
SecureRandom.random_bytes
  Integer#to_int <0.000005>
  SecureRandom.gen_random
    OpenSSL::Random.random_bytes <0.002223>
  SecureRandom.gen_random <0.002243>
SecureRandom.random_bytes <0.002290>

Digest::SHA256.digest
  Digest::Class#initialize <0.000004>
  Digest::Instance#digest
    Digest::Base#reset <0.000005>
    Digest::Base#update <0.000210>
    Digest::Base#finish <0.000006>
    Digest::Base#reset <0.000005>
  Digest::Instance#digest <0.000267>
Digest::SHA256.digest <0.000308>

Kernel#rand
  Kernel#respond_to_missing? <0.000008>
Kernel#rand <0.000071>

Kernel#sleep <1.003233>
```

and with `-m` for singling out a method

```ruby
rbtrace -p 12345 -m digest
*** attached to process 12345
Digest::SHA256.digest
  Digest::Instance#digest <0.000201>
Digest::SHA256.digest <0.000220>

Digest::SHA256.digest
  Digest::Instance#digest <0.000287>
Digest::SHA256.digest <0.000343>
```

or get a heap dump (memory leaks?)

```ruby
bundle exec rbtrace -p <SERVER PID HERE> \
  -e 'Thread.new{GC.start;require "objspace"; \
      io=File.open("/tmp/ruby-heap.dump", "w"); \
      ObjectSpace.dump_all(output: io); io.close}'
```
