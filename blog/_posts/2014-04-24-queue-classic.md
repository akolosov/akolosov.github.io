---
layout: post
title:  "В очередь, сукины дети!"
date:   2014-04-24 10:00:00
tags:
- ruby
- gem
- queue
- workers
---
В одном из моих проектов понадобился простая, быстрая и надёжная очередь сообщений. Сразу оговорюсь - проект не на Rails. Sinatra + PostgreSQL = API-сервер. Первое, что приходит в голову - [Sidekiq](https://github.com/mperham/sidekiq) и [Resque](https://github.com/resque/resque). Ничего не могу про них сказать плохого или хорошего - никогда не использовал... и не планирую пока. Эти продукты мне не подходили. Нужно было что-то простое и понятное. Мне не хотелось поднимать дополнительные сущности вроде Redis'а, RabbitMQ и прочих. У меня уже был PostgreSQL - простой и надёжный как топор. И тут, впрочем как всегда, помог Google - я нашел [его](https://github.com/ryandotsmith/queue_classic) - простой провайдер и обработчик очереди сообщений с PostgreSQL в качестве бэкэнда.

Ставится все по старинке:

{% highlight ruby %}

    gem install queue_classic

{% endhighlight %}

... и добавляется в Gemfile проекта:

{% highlight ruby %}

    gem 'queue_classic'

{% endhighlight %}

Далее создайте таблицу для добавления сообщений в очередь:

{% highlight ruby %}

    bundle exec rake qc:create

{% endhighlight %}

Для worker'а пишется Rakefile:

{% highlight ruby %}

    ENV["QC_DATABASE_URL"] = "postgres://api:ipa@db_host/db_api"
    ENV["QC_PUSH_DEBUG"] = "true" 

    require 'bundler'

    Bundler.require

    require 'queue_classic'
    require 'queue_classic/tasks'

    class Worker

        def self.process_message hash
            if ENV["QC_PUSH_DEBUG"] == "true"
                puts "param1 = #{hash['param1']}"
                puts "param2 = #{hash['param2']}"
                puts "param3 = #{hash['param3']}"
            end
            # do something with message
        end

    end

{% endhighlight %}

Чтобы добавлять сообщения в очередь include'им в приложение класс QC:

{% highlight ruby %}

    module Helpers
        module Messages

            include QC
            
            def put_message_to_queue
                QC.enqueue('Worker.process_message', { 'param1' => 'param1', 'param2' => 'param2', 'param3' => 'param3' })
            end
        end
    end

{% endhighlight %}

Далее параллельно с приложением запускаем воркер или несколько воркеров:

{% highlight ruby %}

    bundle exec rake qc:work

{% endhighlight %}

Количество воркеров ограниченно ресурсами сервера и решаемой задачей.

Как-то так. Все предельно просто и надёжно. Главное решает свою задачу на "отлично".


----------

**Полезности:**

[#344 Queue Classic](http://railscasts.com/episodes/344-queue-classic)

[Getting started with QueueClassic](https://blog.rainforestqa.com/2014-04-17-getting-started-with-queue-classic/)
