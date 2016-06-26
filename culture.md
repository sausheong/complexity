# Programming Complexity

One of the more fascinating new sciences that came up in recent decades (yes, science takes decades) is complexity science. Complexity science is the study of complex systems, systems where many different components interact with one another to produce behaviors that are not easily explained from observing the individual components. An example of this is the way fish swim in the same direction in a coordinated way(i.e. how fish school). From observation and [mathematical modeling][1], fish were simulated to school from individual behaviors that are as simple as:

1. Remain close to your neighbours
2. Avoid collisions with your neighbours

This simple rules however don't tell the story of the spectacular way of how 'barracuda tornadoes' form from thousands of fish as they circle round and round in a vortex to avoid being fixed up as prey in the eyes of the predator.

![Barracuda tornado, Barracuda Point, Sipadan, Malaysia](figures/culture/schooling_fish.jpg)

Schooling fish is an obvious example of a complex system, but there are complex systems all around us. If you observe our daily lives you'd realize that the everyday culture you're used to is likely be a hodge-podge mix of multiple cultures. What you identify as a national or local culture is probably an eclectic mixture of a number of other cultures, which in turn are also a mix of even more other cultures. 

For example, take laksa, a popular spicy noodle soup dish in Singapore, Malaysia, Indonesia and southern Thailand. Its origins are believed to have been from Chinese coastal settlements, which took on a local twist as Chinese immigrants introduced local ingredients and cooking methods. And as local ingredients go, this can be coconut milk (resulting in variants of curry laksa), tamarind (resulting in variants of assam laksa), sambal belacan, various species of fish and even lemongrass! This results in a delightful and dizzying variety of variants that are simply edible embodiment of the local cultural mix.

![Curry laksa](figures/culture/laksa.jpg)

Similarly, we find linguistic equivalents in [Singlish](https://en.wikipedia.org/wiki/Singlish), an English-based creole language spoken in Singapore, containing words originating from English, Malay, various dialects of Chinese and Indian languages, and also in [Bahasa Rojak](https://en.wikipedia.org/wiki/Bahasa_Rojak) a Malaysian pidgin language formed by code-switching between English, Malay, various dialects of Chinese and Indian languages in Malaysia. If we dig deeper, various things we take for granted as being iconic to a culture, often have origins in other cultures for e.g. [English tea](https://www.tea.co.uk/history-of-tea) (which came from China) and [Zen Buddhism](https://en.wikipedia.org/wiki/Zen) (popularized in the West through Japanese Zen, which came from Chan Buddhism, which originated in China and was influenced by Taoism, but in turn is a school of Mahayana Buddhism which came from India).

As Stephen Hawkings wrote in the open to "A Brief History of Time" -- it's turtles all the way down.

![Turtles all the way down](figures/culture/turtles.jpg)

How people and cultures interact and influence each other is the subject of numerous research. Various social scientists, anthropologists and biologists have studied cultural changes, came up with plausible theories and have written papers to describe these theories. One such [paper][2] by Robert Axelrod, a seminal one that has been referenced in much later research, built an agent-based model that follows a set of simple rules in an attempt to produce emergent behavior (that is, he built a model of a complex system).

I've attempted to do an alternate agent-based model based on the rules described on Axelrod's paper. I used Ruby to simulate the model and React to visual it. 

The model I used is based on two very simple premises:

1. Cultures that are similar to each other are more likely to interact that cultures that are not
2. When cultures interact they become more alike each other

Using these two premises, I built a model using a square 36x36 grid. Each cell in the grid represents a culture. 

![Simulation grid, each cell describing a culture](figures/culture/grid1.jpg)

In the simulation, each culture has 6 features, where a feature is an attribute of a culture like food, language, religion etc. 

![Each culture has 6 features](figures/culture/grid2.jpg)

Each feature in turn has 16 different traits, which are the different possible values of the feature.

![Each feature has 16 traits](figures/culture/grid3.jpg)

As you probably realize, this is obviously modeled to make it easier for me to describe the grid in hexadecimal colors.

![hexadecimal colors](figures/culture/color.jpg)

Let's see how the simulation works. Each cell (culture) has 8 neighbors, unless it is a corner or cell.

![Each cell has 8 neighbors](figures/culture/grid4.jpg)

At every interval, the simulation randomly picks n number of cultures. The selected culture is compared with its neighbors, feature to feature. The difference between one culture and another, which I call the cultural difference, is the probability that one culture will influence the other. If there is cultural influence, the simulation chooses a random trait to copy to the other culture, making it more similar to each other.

In the example below, take 2 cultures A and B that are next to each other (i.e. they are neighbors). Now compare them feature to feature. In the figure below, the difference between two features, or feature difference is 3. Note that we're only looking for the difference between the two features, so we are getting the absolute number, and not simple subtracting one number from another.

![Comparing features between culture A and B](figures/culture/culture1.jpg)

Next, we add up all the features differences between the two cultures to get a cultural difference. In the figure below, we get a cultural difference of 34 (if we step back a bit you'd realize that we've just reduced the difference between two cultures to a single number, but that's the nature of modeling).

![Cultural difference between two cultures is 34](figures/culture/culture2.jpg)

A quick arithmetic check wtill tell us that if two cultures are completely different from each other, their cultural difference is 96, and if they are exactly the same, their cultural difference is 0. Therefore, the probability that there will be cultural exchange is:

![Probability of cultural exchange](figures/culture/culture3.jpg)

Finally, if cultural exchange does happen, the simulation will randomly pick one feature and copy the trait value from a culture to another. In this case, culture A's feature 3 is copied to culture B.

![Copy a feature value from one culture to another](figures/culture/culture4.jpg)

Let's take a look at the simulation code next. First, we need to build out the grid using a Grid class, which is subclassed from Array. Here's the code in a file named grid.rb.

```ruby
class Grid < Array
  attr :width
  
  def initialize(size)
    @width = size
    (size*size).times do |i|
      self[i] = Random.rand(0xFFFFFF)
    end    
  end

  def find_neighbours_index(n)
    nb = []
  	case
  	when top_left(n)
  		nb << c5(n) << c7(n) << c8(n)
  	when top_right(n)
  		nb << c4(n) << c6(n) << c7(n)		
  	when bottom_left(n)
  		nb << c2(n) << c3(n) << c5(n)
  	when bottom_right(n)
  		nb << c1(n) << c2(n) << c4(n)
  	when top(n)
  		nb << c4(n) << c5(n) << c6(n) << c7(n) << c8(n)
  	when left(n)
  		nb << c2(n) << c3(n) << c5(n) << c7(n) << c8(n)
  	when right(n)
  		nb << c1(n) << c2(n) << c4(n) << c6(n) << c7(n)
  	when bottom(n)
  		nb << c1(n) << c2(n) << c3(n) << c4(n) << c5(n)
  	else
  		nb << c1(n) << c2(n) << c3(n) << c4(n) << c5(n) << c6(n) << c7(n) << c8(n)
    end
  	nb
  end


  def top_left(n)      n == 0; end
  def top_right(n)     n == @width-1; end
  def bottom_left(n)   n == @width*(@width-1); end
  def bottom_right(n)  n == (@width*@width)-1; end
  
  def top(n)    n < @width; end
  def left(n)   (n%@width) == 0; end
  def right(n)  (n%@width) == @width-1; end
  def bottom(n) n >= (@width*(@width-1)); end
  
  def c1(n) n - @width - 1; end
  def c2(n) n - @width; end    
  def c3(n) n - @width + 1; end
  def c4(n) n - 1; end
  def c5(n) n + 1; end
  def c6(n) n + @width - 1; end        
  def c7(n) n + @width; end
  def c8(n) n + @width + 1; end
      
end

```

When we create an instance of Grid, all its cells are initialized with random colors. The 


```ruby


```


```ruby


```



[1]: http://hdl.handle.net/1828/1322 "Charnell, M. (2008) Individual-based modelling of ecological systems and social aggregations"
[2]: http://www-personal.umich.edu/~axe/research/Dissemination.pdf "Axelrod, R. (1997) The dissemination of culture a model with local convergence and global polarization"