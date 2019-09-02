+++
title = "Ruby'de Delegatorlar"
date = 2019-09-02T10:56:08+03:00
author = "personinblack"
cover = "markus-spiske-xwvVT1XN2HE-unsplash.jpg"
tags = ["ruby"]
keywords = ["ruby", "delegator", "simpledelegator", "decorator", "pattern", "oop", "object", "oriented"]
showFullContent = false
description = """
Delegatorlar bir obje üzerinde kullanılan ve obje tarafında bir karşılığı bulunmayan tüm
metotların constructorda tanımlanmış bir başka objeye yönlendirilmesini sağlar. Bu tanım
belki biraz karmaşık gelmiş olabilir fakat benimle kal, mevzu oldukça basit.
"""
+++

    Fotoğraf Markus Spiske

# Ruby'de Delegatorlar

Delegatorlar bir obje üzerinde kullanılan ve obje tarafında bir karşılığı bulunmayan tüm
metotların constructorda tanımlanmış bir başka objeye yönlendirilmesini sağlar. Bu tanım
belki biraz karmaşık gelmiş olabilir fakat benimle kal, mevzu oldukça basit.

Öncelikle klasik bir örnekle başlayalım:

```ruby
class Pizza
  def slice
    'Dilimlendim!'
  end

  def to_s
    'Ben bir pizzayım!'
  end
end

class PizzaWithPepperoni < Pizza
  def to_s
    super.sub(/((?!\s).)+\!/, 'pepperonili pizzayım!')
  end
end

class PizzaWithMushroom < Pizza
  def to_s
    super.sub(/((?!\s).)+\!/, 'mantarlı pizzayım!')
  end
end

puts PizzaWithPepperoni.new.to_s #=> "Ben bir pepperonili pizzayım!"
puts PizzaWithMushroom.new.to_s #=> "Ben bir mantarlı pizzayım!"

puts PizzaWithPepperoni.new.slice #=> "Dilimlendim!"
```

Yukarıdaki örnekteki classlar ile 3 farklı Pizza üretebiliyoruz:
*Klasik Pizza*, *Mantarlı Pizza* ve *Pepperonili Pizza*. Eğer mantarlı ve pepperonili bir
pizza istersek yeni bir obje oluşturmamız gerekecek. Bir de işin içerisine *Domatesli
Pizza*'nın girdiğini düşün! Bu durumda 3 farklı malzeme ve bu malzemelerle yapılabilecek
bir sürü farklı pizza için bir sürü farklı class...

Böyle bir durumda decorator pattern kullanmayı deneyebiliriz:

```ruby
class Pizza
  def slice
    'Dilimlendim!'
  end

  def to_s
    'Ben bir pizzayım!'
  end
end

class Pepperoni
  def initialize(base)
    @base = base
  end

  def to_s
    @base.to_s.sub(/((?!\s).)+\!/, 'pepperonili pizzayım!')
  end
end

class Mushroom
  def initialize(base)
    @base = base
  end

  def to_s
    @base.to_s.sub(/((?!\s).)+\!/, 'mantarlı pizzayım!')
  end
end

puts Mushroom.new(Pizza.new).to_s #=> "Ben bir mantarlı pizzayım!"
puts Pepperoni.new(Mushroom.new(Pizza.new)).to_s #=> "Ben bir mantarlı pepperonili pizzayım!"
```

Yukarıdaki örnek çok daha iyi öyle değil mi? Malzemeleri istediğimiz gibi ekleyip
çıkartabiliyoruz mesela. Yeni bir malzeme üretmek için ise tek yapmamız gereken yeni bir
class oluşturmak. Böylelikle elimizde sadece `malzeme sayısı + 1` adet class oluyor.

Fakat bu örnekte de bir sıkıntı var. `slice` metotu sadece `Pizza` sınıfına ait. Yani
`Pepperoni.new(Pizza.new).slice` dediğimizde `undefined method 'slice'` hatasını
alıyoruz. Çünkü Pepperoni'de bu metotun bir karşılığı yok.

Peki bunu nasıl çözeriz? Malzemelerde `slice` metotunu tanımlayabiliriz; fakat slice gibi
başka metotlar da olabileceği için bu oldukça zahmetli olur ve normalde çok basit olması
gereken malzeme classlarını karmaşıklaştırabilir. Delegatorlar? Evet Delegatorlar.

Yukarıdaki malzeme classlarını SimpleDelegator kullanacak şekilde düzenleyelim:

```ruby
class Pepperoni < SimpleDelegator
  def to_s
    super.to_s.sub(/((?!\s).)+\!/, 'pepperonili pizzayım!') #=> "Ben bir pepperonili pizzayım!"
  end
end

pizza = Pepperoni.new(Pizza.new)
puts pizza.to_s #=> "Ben bir pepperonili pizzayım!"
puts pizza.slice #=> "Dilimlendim!"
```

SimpleDelegator, Pepperoni objesi üzerinde kullanılan `slice` metotunu, otomatik olarak
constructorda tanımlanan Pizza objesine yönlendirerek, metotun çalışmasını sağladı.
Üstelik daha önce elle tanımlamak zorunda kaldığımız constructor metotunu da kendisi
tanımladığından, classımızı daha minimal bir hale getirebildik.
