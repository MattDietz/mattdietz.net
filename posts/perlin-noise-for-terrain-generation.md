<!-- 
.. title: Perlin Noise for terrain generation
.. slug: perlin-noise-for-terrain-generation
.. date: 2015-02-22 18:52:44 UTC
.. tags: algorithms, games, gamdev, graphics
.. link: 
.. description: 
.. type: text
-->

One of the things I find fascinating is the idea of "random" content generation for games, and lately I've been fascinated with procedural generation of maps and terrain. One of the things that seems to be most prevalent in games is the need for realistic looking terrain. A common technique for this is generating [Heightmaps](http://en.wikipedia.org/wiki/Heightmap) for use in commercial game engines like Unity and Unreal as well voxel based games like Minecraft. The values of the individual pixels of the heightmap are interpreted literally as terrain elevation. The problem with heightmap generation is it's tough to produce natural looking terrain with random numbers alone. This is where Perlin noise comes in.

As you can see, Perlin noise is wonderful for generating things that look like terrain:

![CDN Test Screenshot](http://71736c127cff565b91bf-0044c42a78e82872de2148fe3ea73ce3.r79.cf2.rackcdn.com/terrain_perlin.png)
![CDN Test Screenshot](http://71736c127cff565b91bf-0044c42a78e82872de2148fe3ea73ce3.r79.cf2.rackcdn.com/grayscale_perlin.png)

<!-- TEASER_END -->

I decided I want to slap together a quick Perlin noise generator in Python, using this [How to Use Perlin Noise in Your Games](http://devmag.org.za/2009/04/25/perlin-noise/) as reference. The implementation provided there is in a pseudo-C like language, so I had to set about writing something that would work easily in Python with Pygame. I highly recommend reading that article for a much more in-depth explanation of everything that's going on, but the long and short goes something like:

1. Generate a field of "random" noise

        def generate_base_noise():
            base_noise = []
            grid_width = CONF["width"] / CONF["cell_size"]
            grid_height = CONF["height"] / CONF["cell_size"]
            for i in xrange(grid_height):
                base_noise.append([])
                for j in xrange(grid_width):
                    base_noise[i].append(random.random())
            return base_noise


2. "Smooth" the initial random field multiple times into a collection of fields with the same dimensions as the original, where each generation is more "blurred" than the previous (Think of the Droplet tool in Photoshop)

        def generate_smooth_noise_at_octave(base_noise, octave):
            smooth_noise = []
        
            # Did you know bitshift actually worked in Python?
            sample_period = 1 << octave
            sample_frequency = 1.0 / sample_period
        
            grid_width = CONF["width"] / CONF["cell_size"]
            grid_height = CONF["height"] / CONF["cell_size"]
        
            for i in xrange(grid_height):
                smooth_noise.append([])
                sample_i0 = int((i / sample_period) * sample_period)
                sample_i1 = int((sample_i0 + sample_period) % grid_height)
                vertical_blend = (i - sample_i0) * sample_frequency
        
                for j in xrange(grid_width):
                    sample_j0 = int((j / sample_period) * sample_period)
                    sample_j1 = int((sample_j0 + sample_period) % grid_width)
                    horizontal_blend = (j - sample_j0) * sample_frequency
        
                    top = lerp(base_noise[sample_i0][sample_j0],
                               base_noise[sample_i0][sample_j1], horizontal_blend)
        
                    bottom = lerp(base_noise[sample_i1][sample_j0],
                                  base_noise[sample_i1][sample_j1], horizontal_blend)
        
                    smooth_noise[i].append(lerp(top, bottom, vertical_blend))
        
            return smooth_noise

3. Blend all of the "smoothed" fields together

        def generate_perlin_noise(smooth_noise):
            blended = []
            amplitude = 1.0
            total_amplitude = 0.0
        
            grid_width = CONF["width"] / CONF["cell_size"]
            grid_height = CONF["height"] / CONF["cell_size"]
        
            for i in xrange(grid_height):
                blended.append([])
                for j in xrange(grid_width):
                    blended[i].append(0.0)
        
            for octave in xrange(CONF["octaves"]-1, -1, -1):
                amplitude *= CONF["persistance"]
                total_amplitude += amplitude
                for i in xrange(grid_height):
                    for j in xrange(grid_width):
                        blended[i][j] += smooth_noise[octave][i][j] * amplitude

This is not my best Python code. I just wanted something to play with :-)

The above images are examples of output from my Python code, and you can find the source [Here](https://github.com/Cerberus98/python_noise/blob/master/perlin.py)
