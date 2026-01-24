# Godot Bite Sized

My name is Tim and I am a hobbyist game designer / developer. I've been using Godot for about a year and a half, on and off, to develop some fun little game ideas that I've had in my head. I wanted to share what works for me and how I go about building some games using the Godot Engine.

The ideas that I'm going to capture here are targeted to be less than 7 minutes worth of work, or bit sized. The reading might be a bit longer, but the actual work in Godot should be no more than 7 minutes. One of my goals is to also make these sections into videos and I am starting with text-based solutions first.  

## How I Think About Building Games

Game Design is not why we're here. It's to get into Godot. But, I don't believe you can jump right into Godot and start building your game without thinking and designing it. Well, maybe you can. I can't.

My first step in building games is to think about what type of game I'm interested in building. Inspiration comes at strange times, I'll write the idea down and come back to it later. Eventually, an idea will grab me and I'll start to work through it. Will it be fun? I'll see as I get through it!

There are many great books on game design. Two that quickly come to mind are A Theory of Fun for Game Design by Raph Koster and The Role of a Great Game Designer by Richard Carillo. I try to follow their guidance from the books as I work through the design. I suggest you read a few.

I also like to think of the lore of the game world that I'm thinking of at this time. Is it modern? Fantasy? Why will a player care? I like writing the story of the world and thinking where players can take themselves!

## Who is this geared towards?

This content is targeted towards Godot high beginners to low intermediate developers. If you are a true beginner, some of the concepts here might seem more complicated than just building a game demo. I'd highly recommend the [Official Godot Tutorials](https://docs.godotengine.org/en/stable/getting_started/step_by_step/index.html) or [Godot Tutorials Basic Series](https://godottutorials.com/courses/godot-basics-series). You'll pick up a lot from them.

If you are an advanced developer, some of the starting portions of the components might be very basic. In the start of each section, I'll point out what is being built and you can skim over it to see if there's anything new for you or not. Hopefully you will find some nuggets of knowledge or maybe you can share yours via issues!

## Now Into Godot!

Great, there's an idea of a game! Once I get that idea rolling, I start to think about the components of my game. Will the player have an inventory? Is it fixed size or just a bag of holding? What type of gear will be needed? What is the world like? What do I need to build first so that I can get this idea rolling?

That last question is really the big one. What to build first. When that's answered, I start up Godot and get to building that first subsystem. My thought is to get something working to test out if the idea is fun! After all, we are making a game and my games typically focus on fun as the feeling.

As with most developers, I don't like to build things more than once. Ask a game developer how many chat systems they've developed. The answer is either "one" or "too many." If I build something once, I try to make it a component that can be re-used in future games. I do this by creating a separate project and working on the interfaces to and from the component. My first goal is always to get something working. I might have an idea of how, and a design that I'm working from, but I will adjust when I see what is or is not working.

Once I get something working, I typically spend some time cleaning it up. I like my components to have minimal code in each object and to be maintainable. But this is _only_ if they work. Optimizing something that doesn't work is, to me, a fool's errand.

When I get the component to a spot that I like, I copy it into the main game and see what I like and don't like about how it works. This means that I will get some form of each of the portions of a game working and start seeing how they work together. Did I get it right? Did I over-simplify or create an overly complex component? Did I forget something super obvious? You'll see some of that in somewhat real-time!

Godot experts might wonder why I say components and not plug-ins. I just haven't gotten that far with my components. I have developed some plug-ins for other usage, but not in my own code base. One day, that might change. It might even change before your eyes!

Other Godot experts might wonder if I will use plug-ins in this. The quick answer is "No." The longer answer is I want to help people understand what they are doing and to not just use plug-ins. I think plug-ins have a purpose and some are very powerful. If you don't want to be a developer, but you love putting games together from building blocks, plug-ins can be a wonderful way to get that moving! Additionally, I've learned from some plug-ins by reviewing their code and looking at what the developer(s) have done. I love learning from others and do not need to learn every lesson the hard way!

## OK, nice readme, but what's on the roadmap?

I want to build a game of an idea that I've had for a while. It's going to need some components, namely:

- A Player
- Enemies
- Boss enemies
- A dynamic floor/room generator for a 2D, top-down world
- An inventory system, including a paperdoll to equip items
- Ranged weapons with damage types
- An interesting armor system that makes players make choices
- A dynamic loot table, with tiers and qualities
- NPCs that offer rewards for accomplishments

There's probably a few other things that will get added to this list. The way that I will build this out is by using different repos for each of the major items and link the repo on this page.

Have fun and happy gaming!

## Great! Where do we start!

Here is the order of content released. This will probably bounce around a bit, from component to component. Feel free to follow a component all the way through, if you like. I realize with 1 demo here, that's pretty easy :D

### Basic Build

1. [Player Build 1](https://github.com/timjbruce/godotbitesized_player/blob/main/readme.MD#build-1) - 1a16275e3b4470524be13ed08eeed15e9c34b053
2. [World Build 1](https://github.com/timjbruce/godotbitesized_world/blob/main/README.md#build-1) - a722a2dde57437ad26b300bf1d14f4c9902e2948
3. [Generator Build 1](https://github.com/timjbruce/godotbitesized_generator/blob/main/README.md#build-1) - 13fae4a34aafd3e9e1478429efff72015360dedb
4. [World Build 2](https://github.com/timjbruce/godotbitesized_world/blob/main/README.md#build-2) - 0aea30cbacb6c57dff278fcccbb465d563a14559

### Adding Basic Physics

5. [Generator Build 2](https://github.com/timjbruce/godotbitesized_generator/blob/main/README.md#build-2) -c09a800bd7a72e74315909568aa06c5ce81d3ce8
6. [Player Build 2](https://github.com/timjbruce/godotbitesized_player/blob/main/readme.MD#build-2) - 31e471601c13e17e91e04ffa4b73f3743f1381aa
7. [World Build 3](https://github.com/timjbruce/godotbitesized_world/blob/main/README.md#build-3) - d03bdca8e03a8e8786fa6d7060f1cde36af046f1

### Adding Features - Player Component

8. [Player Build 3](https://github.com/timjbruce/godotbitesized_player/blob/main/readme.MD#build-3)
9. [World Build 4](https://github.com/timjbruce/godotbitesized_world/blob/main/README.md#build-4)

## Asked Questions

I won't say these are **_Frequently_** Asked Questions. These are just some notes of who I am and what you might care about.

1. What do you do for your day job?

During my day, I help customers with cloud-based solutions and use scripting languages, such as Node and Python, fairly often. I like to see results quickly and jump through debugging quickly.

2. Why should we pay attention to you?

I like to figure out how things work. If you want to know how things work and how to get them to work yourself, you might find some knowledge here. Heck, maybe even open an issue and let me know what you're working on!

3. This seems very beginner based. Is it?

Some pieces will be beginner based, especially the early builds of the components. I'm going to document the build of these features in bit sized chunks that can help the beginner to advance. Hopefully advanced users can learn something, too. If you have ideas on any of the repos, please open an issue with your idea. I can't promise that I will build what you are asking about, but I will engage!

4. Ok, you're making component repos and building up bit by bit. How can we see each stage if you're always updating a repo?

I'm going to tag each step that is documented in the repo. You can checkout a specific tag to see the code that completes the issue. If you want to work along, checkout the prior tag and follow along!

5. You mean I need to learn git?

All game developers should use some form of code repository/code management system. If you want to be a game developer, learn a bit of how to keep your code safe from your own local disasters!

6. Aren't you afraid of putting your work in progress out there?

No. I'm here to have fun and learn. I hope you are, too.

7. Are you an artist?

No, I am not. My graphics might be ugly to some. Some will definitely be bland. I may use some open source graphics just to save time on building content. I realize this might sound a bit like I'm speaking out of two sides of my mouth with not using plug-ins. I'm a developer and designer, and that's where I love to learn how things work.

8. I have some artwork I'd love to contribute. How?

If you are looking to contribute art, let me know via an issue. You must provide me an open license to use it for this component series. I will not use your artwork in a game I release without your explicit permission.

9. What types of games do you like?

I like a lot of different games - CRPGs, Adventure, City Builders, Rogue/Rogue-Lite/Rogue-Like, Racing. I grew up in a great time in this world - Raph Koster, Sid Meier, Brian Reynolds, Roberta Williams, Danielle Bunten Berry, Chris Sawyer, Will Wrigth, Chris Roberts, Al Lowe, Richard Garriott, American McGee, Alexey Pajitnov, Toru Iwatani, Brad McQuaid, Takshi Iizuka, Eugene Jarvis, Richard Bartle, and Douglas Adams painted my early world with their wonderful imaginations with their wonderful creations.

There are many more great names to add to this list from more recent years, and this is not to slight them in the least. 