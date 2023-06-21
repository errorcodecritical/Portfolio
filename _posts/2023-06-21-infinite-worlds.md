---
layout: post
title:  "Infinite worlds with procedural generation!"
date:   2023-06-21 00:42:26 +0100
categories:
---

Ever wonder how games like *Minecraft*, *Astroneer* or *No Man's Sky* create those incredible open-world settings? Entire solar systems yet to be explored, resources just waiting to be utilized! Undoubtedly it wouldn't be very practical to store all of that data on disk, so how do they do it? The answer, as you might have guessed by the title - **procedural generation**!

Now that probably sounds a little daunting to some of you, but really its super simple, and in this post we're going to explore some working examples that you can play around with yourself.

I will be using the [Godot game engine][godot-engine] and C# for the following examples, but please note that these concepts can be implemented in any engine of your choosing.

# Where do we even begin?

The first step towards creating our terrain is to figure out how we're going to represent it in space. Taking some inspiration from *Minecraft*, lets make a grid of uniformly-spaced cubes and build a blocky landscape!

Since we're going to be using cubes for our terrain, we'll first need some a way to create them! In this case, I created `MeshInstance3D` with the default `BoxMesh`, and saved it in its own scene.

This lets us to load it wherever we want like so:


```c#
using Godot;
using System;

public partial class world : Node3D {
    public PackedScene BlockMesh = ResourceLoader.Load<PackedScene>("res://block.tscn");
    ...
}
```

And then to actually instantiate the block, we can do:

```c#
MeshInstance3D block = BlockMesh.Instantiate() as MeshInstance3D;
```

Great! We now have a way to create blocks and add them to our scene, the next step is to create a grid of these blocks on the XZ plane. To do this, we can use nested for loops to iterate over all the possible grid coordinates:

> Note: In most game engines the Y axis represents the vertical coordinate, which is why we use X and Z instead.  

```c#
for (float x = -20; x < 20; x++) {
    for (float z = -20; z < 20; z++) {
        MeshInstance3D block = BlockMesh.Instantiate() as MeshInstance3D;
        block.Position = new Vector3(x, 0, z);
        AddChild(block);
    }
}
```

<p align="center"><img src="/assets/images/img-002.png"  width="70%" height="70%"></p>

Which should generate something that looks like this, albeit is a little underwhelming - but we're just getting started!

# Bring on the curves!

In order to see anything even remotely interesting, we're going to create a function that'll calculate a new Y coordinate, given X and Z as inputs. For example:

```c#
float GetHeightAtPosition(float x, float z) {
    return (
        MathF.Sin(0.5f * x) + 
        MathF.Sin(0.5f * z)
    );
}
```

And then if we extend our previous code like so:

```c#
float y = GetHeightAtPosition(x, z);
block.Position = new Vector3(x, y, z);
```

<p align="center">
<img src="/assets/images/img-003.png"  width="70%" height="70%">
</p>

We end up with something that somewhat resembles bumpy terrain. Progress!
If you want, you can experiment with different height functions and see what happens. Here are a couple more examples:

```c#
// left image:
y = 0.1 * (x * x + z * z);
// right image:
y = 0.1 * (x - z) * (x + z);
```

<p align="center">
<img src="/assets/images/img-004.png"  width="49%" height="49%">
<img src="/assets/images/img-005.png"  width="49%" height="49%">
</p>

Notice how in both of these examples there are gaps between the blocks, which isn't very natural looking. To fix this, we can manipulate the size of each block, such that we form a solid chunk:

> Note: Since our function can generate negative values, we add an offset height in order to avoid funky behavior when setting the block scale. 

```c#
float y = GetNoiseAtPosition(x, z); 
float offset = 50;

MeshInstance3D block = BlockMesh.Instantiate() as MeshInstance3D;
block.Position = new Vector3(x, offset + y / 2, z);
block.Scale = new Vector3(1, offset + y, 1);

AddChild(block);
```

<p align="center">
<img src="/assets/images/img-006.png"  width="70%" height="70%">
</p>

Which should look something like this - resulting in a smoother, solid chunk.

# Okay, but how to terrain?

Everything we've done so far has been setting up for what's about to come. We're ready to finally tackle *procedural terrain*.

While there are a variety of methods that could be used, one of the most prominent is called [simplex noise][simplex-noise]. Simplex noise can be used to generate a smooth 2D heightmap, given our coordinates (X, Z) which sounds awfully like something we've already implemented, **GetHeightAtPosition()**.

Most game engines come with noise libraries pre-installed, but there are open-source implementations should you find yourself unlucky.

In Godot, we can use the [FastNoiseLite][godot-noise] library, like so:

```c#
public partial class world : Node3D {
    public FastNoiseLite Noise = new FastNoiseLite();
    ...
}
```

And then if we modify our height function:

```c#
float GetHeightAtPosition(float x, float z) {
    return 50 * Noise.GetNoise2D(x * .4f, z * .4f);
}
```

<p align="center">
<img src="/assets/images/img-007.png"  width="70%" height="70%">
</p>

It gives us something that looks like this. Fantastic! 

> Note: the generated chunk might not look the exact same, depending on the noise library that you use.

If you would like more information on noise and how it can be used to generate more complex terrain, I highly recommend you visit this [amazing website][redblobgames-noise].


# Neat, so what's next?

In this post we went over a quick and dirty way to create convincing terrain! In the next one we'll go deeper into how we can refine our code, and how we can generate our terrain chunks at a time!

Until then, stay tuned!

[godot-engine]: https://godotengine.org/
[simplex-noise]: https://en.wikipedia.org/wiki/Simplex_noise/
[godot-noise]: https://docs.godotengine.org/en/stable/classes/class_fastnoiselite.html
[redblobgames-noise]: https://www.redblobgames.com/maps/terrain-from-noise/