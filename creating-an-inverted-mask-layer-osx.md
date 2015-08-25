# Creating an inverted mask layer

* Also known as removing a path from layer.

There are several ways to accomplish this. In fact here are 5 of them: [http://www.cocoawithlove.com/2010/05/5-ways-to-draw-2d-shape-with-hole-in.html](http://www.cocoawithlove.com/2010/05/5-ways-to-draw-2d-shape-with-hole-in.html). Most solutions boil down to:

* Create the whole path yourself and wrap around (this is a pain and can bug out if there are multiple regions like a star)
* Do everything using a CGImageRef (this is just slow)
* Use cool tricks with "even-odd" rendering (this would work great if there weren't multiple overlapping regions)
* Just use rectangles (obvious limitations)

My solution uses a subclass of a CAShapeLayer.

When you subclass a CAShapeLayer the most difficult challenge is getting it to display correctly. For a subclass to call `drawInContext` you must set the `bounds`. When you set the `bounds` the position freaks out. When you want to update... you need to update the `bounds` (which means you have to *change* the value).

Keeping this in mind, I created a subclass:

    @interface CAInvertedMaskLayer: CAShapeLayer
    
    @property (nonatomic, assign) CGPathRef invertedMask;

    @end

    @implementation CAInvertedMaskLayer

    - (void)drawInContext:(CGContextRef)ctx {
		// Custom drawing code goes here
        NSColor *color = [NSColor redColor];
        
        // Draw the path of this layer first (draws everything)
        CGContextSetFillColorWithColor(ctx, color.CGColor);
        CGContextAddPath(ctx, self.path);
        CGContextFillPath(ctx);

		// Clear out the inverted portion
        if (self.invertedMask) {
            CGContextSetBlendMode(ctx, kCGBlendModeClear);
            CGContextSetFillColorWithColor(ctx, [NSColor clearColor].CGColor);
            CGPathRef path = self.invertedMask;
            CGContextAddPath(ctx, path);
            CGContextFillPath(ctx);
        }
    }

    - (BOOL)needsDisplayOnBoundsChange {
        return YES;
    }

    @end

So far so good. Unfortunately when I added my special drawing code, the inherited `CAShapeLayer` was still performing some actions, because of this you want to make sure you haven't set things like `fillColor` on the layer to the wrong value. Generally you want the layer `fillColor` to be `clearColor` otherwise the blending may be confusing.
    
    self.regionLayer = [[CAInvertedMaskLayer alloc] init];
    self.regionLayer.fillColor = [NSColor clearColor].CGColor;

You have to set the position when we override `drawInRect`. Here I am just setting the position based on the bounds of the container (in this case my view).

	// Set the position, because the origin defaults to center, center
    [self.regionLayer setPosition:CGPointMake([self bounds].size.width/2, [self bounds].size.height/2)];

At this point you can simply set the `path` and update the `invertedMask` path.

    CGPathRef regPath = CGPathCreateWithRect(rect, &CGAffineTransformIdentity);
    self.regionLayer.path = regPath;
    CGPathRelease(regPath);

    CGMutablePathRef occlusionPath = CGPathCreateMutable();
    CGAffineTransform transform = CGAffineTransformMakeScale(1, -1);
    transform = CGAffineTransformTranslate(transform, 0, - self.bounds.size.height);
    CGPathAddRect(occlusionPath, &transform, bounds);
    self.regionLayer.invertedMask = occlusionPath;
    CGPathRelease(occlusionPath);

Now the tricky part: to make the render happen you need to set the bounds of the layer. Again, my layer is full width so I am just using the bounds of my view. I set it to an empty rectangle and set it back.

    self.regionLayer.bounds = CGRectNull;
    self.regionLayer.bounds = self.bounds;

# Future directions

Instead of using a path for the `invertedMask` you could use a whole `invertedMaskLayer`. Instead of rendering the path in your `drawInRect` code you could render the whole layer as clear. Using this you could end up with much more advanced masks.