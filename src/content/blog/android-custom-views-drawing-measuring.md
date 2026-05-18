---
heroImage: '/android-custom-views-drawing-measuring.svg'
title: 'Custom Views in Android: Drawing, Measuring, and Layout'
description: 'Step outside standard Android UI widgets. Learn how to create highly performant and unique Custom Views from scratch.'
pubDate: 'May 17 2026'
---

While Android provides a vast library of standard UI components (`TextView`, `RecyclerView`, etc.), sometimes a unique design requires building a view entirely from scratch. Building Custom Views involves hooking into Android's rendering pipeline.

## The View Lifecycle

Creating a custom view requires overriding specific methods to tell the Android system how big the view should be and what it should look like:

1. **`onMeasure()`:** Determine the size of the view.
2. **`onLayout()`:** Assign sizes and positions to child views (only necessary if extending `ViewGroup`).
3. **`onDraw()`:** Render the visual content onto a Canvas.

## 1. Measuring: `onMeasure()`

The parent layout passes `MeasureSpec` constraints (e.g., EXACTLY 100dp, AT_MOST 200dp) to your view. You must calculate your desired size and call `setMeasuredDimension()`.

```kotlin
override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
    val desiredWidth = 300 // calculated based on content
    val desiredHeight = 300

    val widthMode = MeasureSpec.getMode(widthMeasureSpec)
    val widthSize = MeasureSpec.getSize(widthMeasureSpec)

    val finalWidth = when (widthMode) {
        MeasureSpec.EXACTLY -> widthSize // Parent says "Be exactly this big"
        MeasureSpec.AT_MOST -> min(desiredWidth, widthSize) // Parent says "Up to this big"
        else -> desiredWidth // Unspecified, be whatever size
    }

    // Repeat for height...
    
    setMeasuredDimension(finalWidth, finalHeight)
}
```

## 2. Drawing: `onDraw(Canvas)`

This is where the magic happens. You use a `Canvas` (the surface) and a `Paint` (style, color, stroke) to draw shapes, text, and bitmaps.

**Crucial Rule:** `onDraw` is called repeatedly (e.g., during animations). **Never allocate objects inside `onDraw`**. Pre-allocate `Paint` and `Path` objects in the constructor or `onSizeChanged`.

```kotlin
class PieChartView @JvmOverloads constructor(
    context: Context, attrs: AttributeSet? = null, defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {

    // Pre-allocate Paint object
    private val paint = Paint(Paint.ANTI_ALIAS_FLAG).apply {
        style = Paint.Style.FILL
        color = Color.BLUE
    }

    private val rectF = RectF()

    override fun onSizeChanged(w: Int, h: Int, oldw: Int, oldh: Int) {
        super.onSizeChanged(w, h, oldw, oldh)
        // Update bounds only when size changes
        rectF.set(0f, 0f, w.toFloat(), h.toFloat())
    }

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        // Draw a blue arc from 0 to 90 degrees
        canvas.drawArc(rectF, 0f, 90f, true, paint)
    }
}
```

## Handling Touch Events

To make your view interactive, override `onTouchEvent(MotionEvent)`.

```kotlin
override fun onTouchEvent(event: MotionEvent): Boolean {
    when (event.action) {
        MotionEvent.ACTION_DOWN -> {
            // Handle press
            return true // Return true to indicate we consumed the event
        }
        MotionEvent.ACTION_MOVE -> {
            // Handle drag
        }
    }
    return super.onTouchEvent(event)
}
```

By mastering the Canvas and measuring phases, you can create performant, visually stunning components that standard layouts simply cannot achieve.
