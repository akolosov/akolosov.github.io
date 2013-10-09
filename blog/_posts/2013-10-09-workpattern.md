---
layout: post
title:  "Делу - время, потехе - час"
date:   2013-10-09 12:00:00
tags:
- ruby
- gem
- develop
---
Время самое ценное, что у нас есть и его учет - это очень важно! Немного пафосно, но от реальности не убежишь. :) Так вот... В одном из проектов мне понадобился учет рабочего времени. Т.е. не просто "пришел-ушел-прогулял", а в плане планирования рабочего времени на выполнение той или иной задачи. Например: на выполнение очень важной задачи нам отвели 10 часов, а рабочий день у многих из нас, скажем, 8 часов и еще бывают выходные (что это?!). Хотя многие "руководители" и "очень важные начальники" считают, что 24 часа, а то и 36! И никаких выходных! :) Как нам выставить конечный срок выполнения задачи, если она поступила к нам, скажем, в пятницу (ура!) в 16:45 (троекратное ура!)? Можно, конечно, изобрести велосипед и написать функцию с кучей условий (обед? вечер? ночь? выходные?), которая будет высчитывать конечный срок исполнения из полученных начала выполнения и времени на выполнение (в часах или минутах). Я, признаться, собирался именно так и поступить. И тут, совершенно случайно, в ленте твиттера проскочил пост от [@rubygems](https://twitter.com/rubygems) о том, что он рад мне помочь в этом - есть (оказывается) такой gem [workpattern](https://github.com/callenb/workpattern). Он умеет все это делать в лучшем виде!

Все объясняет простой пример кода:

{% highlight ruby linenos %}

  # пишем инициалайзер (например config/initializers/workpattern.rb), ибо паттерны именованые и создаются единожды при старте приложения
  # разумеентся их можно создавать в runtime, но не забывайте их удалять, иначе получите exception при повторном создании

  # создаем новый паттерн
  MyCoolApp::Application.config.workpattern = Workpattern.new('8х5 Workpattern', 2013, 01)
  # выходные у нас есть, что бы кто не говорил!
  MyCoolApp::Application.config.workpattern.resting(days: :weekend)
  # ночью (с 00:00 до 08:59) мы спим (ага!)
  MyCoolApp::Application.config.workpattern.resting(days: :weekday, from_time: Workpattern.clock(0, 0), to_time: Workpattern.clock(8, 59))
  # в обед (с 13:00 до 13:59) мы обедаем (ваш КО)
  MyCoolApp::Application.config.workpattern.resting(days: :weekday, from_time: Workpattern.clock(13, 0), to_time: Workpattern.clock(13, 59)) 
  # вечером (с 18:00 до 23:59) решаем личные проблемы
  MyCoolApp::Application.config.workpattern.resting(days: :weekday, from_time: Workpattern.clock(18, 0), to_time: Workpattern.clock(23, 59)) 

  # бла-бла-бла...

  # простой хелпер для высчитывания конечного срока
  def get_datetime start_datetime, duration, all_arround_day = false
    unless all_arround_day # класс обслуживания НЕ круглосуточный (8х5)
      return MyCoolApp::Application.config.workpattern.calc(start_datetime, duration.minutes)
    else # класс обслуживания круглосуточный (24х7)
      return (start_datetime + duration.hours)
    end
  end

  # бла-бла-бла...

  # GET /incedents/new
  def new
    @incedent = Incedent.new

    # бла-бла-бла...

    if params[:service_class_id]
      # у нас присутствует класс обслуживания с указанными заранее параметрами срока исполнения и т.п.
      @incedent.service_class = ServiceClass.find(params[:service_class_id])

      # бла-бла-бла...

      # performance_hours - количество часов на выполнение в данном классе обслуживания
      # all_around_day = true - круглосуточно
      @incedent.finish_at = get_datetime(DateTime.now, @incedent.service_class.performance_hours, @incedent.service_class.all_around_day)
    else
      @incedent.finish_at = get_datetime(DateTime.now, 8) # по умолчанию на выполнение 8 рабочих часов
    end

    respond_to do |format|
      format.html # new.html.erb
    end
  end

{% endhighlight %}

Я полагаю дальнейшие объяснения излишни, все и так ясно из примера кода. Повторюсь, что создавать статичные паттерны лучше в инициалайзере, но если есть необходимость можно клепать их прямо в runtime.

Удачного программирования!
