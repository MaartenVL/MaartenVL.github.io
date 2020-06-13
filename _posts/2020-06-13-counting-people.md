---
defaults:
  # _posts
  - scope:
      path: ""
      type: posts
    values:
      layout: single
      author_profile: false
      read_time: true
      comments: true
      share: true
      related: true
author: Maarten Van Loo
title: "Counting People"
classes: wide
sidebar:
  - title: "Counting People"
    image: /assets/images/flock_1.gif
    nav: "docs"
---


# Are we following people or just counting them?

A couple of days ago `De Tijd` posted following article on their frontpage. The belgian coastline will be counting people, to assess with regards to COVID-19 where places would become too overcrowded.
If a place becomes too crowded, they could shut it down and avoid a higher risk of virus transmission:

<html>
<blockquote class="twitter-tweet"><p lang="nl" dir="ltr">Is dit wettelijk ok? Betwijfel ik. Commissie Binnenlandse Zaken en privacycommissie moeten zich hier minstens over buigen. <a href="https://t.co/uW7hDmYR4i">https://t.co/uW7hDmYR4i</a></p>&mdash; Theo Francken MP (@FranckenTheo) <a href="https://twitter.com/FranckenTheo/status/1270588989926445058?ref_src=twsrc%5Etfw">June 10, 2020</a>  cards=hidden</blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</html>

This article was picked up by politicians and followers. They feared 1984-esque situations were their freedom would be impaired:

> "Is this legal?"

> "They won't control us!"

> "The communists are at our doorstep!"

Yet, the article  states, that the mechanism of monitoring consist of `counting people that pass over a line.`
The newspaper unfortunately used a rather "clickbaity" title:  they mention "following people", were the company will be only "counting" them, in an anonymous way.
The same day, the national radio station `Radio 1` had an interview on the monitoring system, where it was also specifically said they would not use any face recognition.

The bottomline is, that the counting of people is done in a `sensor-like way`:

> The detection of people is nothing more than a change in color at a particular spot on the surveillance camera.

The following animation illustrates this. It's an abstraction from the real problem, but the core principle will be the same. Basically, somewhere on the surveillance camera's there will be a reference line.
In the example below, this is indicated by the black area at the top. The `P5.js` script running this animation is always looking for a change in color of that black line. I kept track of the total amount of pixels that had a change in value,
and this is represented by the number in the top left corner. The higher the value, the more the black area is occupied by moving balls. Or, the higher that number, the more people in that specific zone of the canvas.

**The balls are generated randomly each time you refresh the page. If they get stuck, just refresh again to initiate a new set of balls at slighly different positions!**

There are several ways to count people through camera's. I can imagine that there will always be some fear of being watched over, and that if this data would come in hands of the wrong people,
with wrong ideas, it could be used for lesser good. However, I hope to show that it's perfectly possible to use technology for the benefit of society. 

<html>
<iframe src="https://editor.p5js.org/maartenvanloo/embed/LwX9QPQlr" style="width: 800px; height: 400px; overflow: hidden;"  scrolling="no" frameborder="0" ></iframe>
</html>

The code was based on this [example on the P5.js website](https://p5js.org/examples/motion-bouncy-bubbles.html):

```javascript
let numBalls = 20;
let spring = 0.5;
let gravity = 0.001;
let friction = -0.9;
let balls = [];

function setup() {
  createCanvas(800,400);
  for (let i = 0; i < numBalls; i++) {
    balls[i] = new Ball(
      random(width),
      random(height),
      random(30, 70),
      i,
      balls
    );
  }
  noStroke();
  fill(10, 204);
}

function draw() {
  background(255);
  
        fill(0,0,0);
      rect(0, 0, 800, 100);
  
  balls.forEach(ball => {
  fill(180,180,180,120);

    ball.collide();
    ball.move();
    ball.display();
            
  });
    let c = get(0, 0, 800, 100);

 //print(c)
    loadPixels();
  var lls=0
  for (var x =  0; x < 800; x+=1) {
    for (var y = 0; y < 100; y+=1) {
      var loc = (x + y * c.width) * 4;
      var h = pixels[loc];
      var s = pixels[loc + 1];
      var l = pixels[loc + 2];
      var a = pixels[loc + 3];
      ll = int(l);
      ss = int(s);
      hh = int(h);
      //histogram[b]++
      lls = (lls + ll)

    }
  }
  percentage = lls/160000
    fill(180,180,180,255);

  text(str(round(percentage)),50,70)
    textSize(25);

  text("Higher number = more people in the black zone!",50,25)
    textSize(50);

}

class Ball {
  constructor(xin, yin, din, idin, oin) {
    this.x = xin;
    this.y = yin;
    this.vx = 0;
    this.vy = 0;
    this.diameter = din;
    this.id = idin;
    this.others = oin;
  }

  collide() {
    for (let i = this.id + 1; i < numBalls; i++) {
      // console.log(others[i]);
      let dx = this.others[i].x - this.x;
      let dy = this.others[i].y - this.y;
      let distance = sqrt(dx * dx + dy * dy);
      let minDist = this.others[i].diameter / 2 + this.diameter / 2;
      //   console.log(distance);
      //console.log(minDist);
      if (distance < minDist) {
        //console.log("2");
        let angle = atan2(dy, dx);
        let targetX = this.x + cos(angle) * minDist;
        let targetY = this.y + sin(angle) * minDist;
        let ax = (targetX - this.others[i].x) * spring;
        let ay = (targetY - this.others[i].y) * spring;
        this.vx -= ax;
        this.vy -= ay;
        this.others[i].vx += ax;
        this.others[i].vy += ay;
      }
    }
  }

  move() {
    this.vy += gravity;
    this.x += this.vx;
    this.y += this.vy;
    if (this.x + this.diameter / 2 > width) {
      this.x = width - this.diameter / 2;
      this.vx *= friction;
    } else if (this.x - this.diameter / 2 < 0) {
      this.x = this.diameter / 2;
      this.vx *= friction;
    }
    if (this.y + this.diameter / 2 > height) {
      this.y = height - this.diameter / 2;
      this.vy *= friction;
    } else if (this.y - this.diameter / 2 < 0) {
      this.y = this.diameter / 2;
      this.vy *= friction;
    }
  }

  display() {
    ellipse(this.x, this.y, this.diameter, this.diameter);
  }
}

```

