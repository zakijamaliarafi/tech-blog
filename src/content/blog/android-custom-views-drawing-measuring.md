---
heroImage: '/android-custom-views-drawing-measuring.svg'
title: 'Custom Views in Android: Drawing, Measuring, and Layout'
description: 'Step outside standard Android UI widgets. Learn how to create highly performant and unique Custom Views from scratch.'
pubDate: 'May 17 2026'
---

The Android SDK provides developers with an incredibly robust and diverse library of pre-built user interface components. From the ubiquitous `TextView` and `Button` to highly complex, scrolling architectures like `RecyclerView` and `ConstraintLayout`, you can construct the vast majority of modern application interfaces by simply combining and styling these standard widgets. 

However, there inevitably comes a time in every senior Android developer's career when the standard library falls short. Perhaps your design team has handed you a mockup featuring a highly interactive, animated radial dial, a complex statistical graphing component with intricate data visualizations, or a completely custom calendar grid that standard layouts cannot render efficiently. When you hit the limitations of XML styling and subclassing standard widgets, you must step into the realm of **Custom Views**.

Building a Custom View from scratch means bypassing the high-level XML layout system and hooking directly into the lower-level Android rendering pipeline. It requires a fundamental understanding of how the Android system calculates dimensions, positions elements on the screen, and physically pushes pixels to the display hardware. 

This comprehensive guide will demystify the Custom View lifecycle, focusing heavily on the critical trio of operations: Measurement (`onMeasure`), Layout (`onLayout`), and Drawing (`onDraw`).

## Understanding the View Lifecycle

Before writing a single line of code, you must understand the sequence of events that occurs when an Android Activity is rendered. When a layout is inflated, the Android framework traverses the View Hierarchy tree (starting from the root `ViewGroup` down to the lowest leaf `View` nodes) in a specific, strictly ordered lifecycle:

1.  **Instantiation:** The View is created. If inflated from XML, the constructor taking a `Context` and an `AttributeSet` is invoked, allowing you to parse custom XML attributes.
2.  **`onMeasure(widthMeasureSpec, heightMeasureSpec)`:** The framework asks the View, "Based on these constraints from your parent, how big do you want to be?"
3.  **`onSizeChanged(w, h, oldw, oldh)`:** Invoked immediately after the size of the view is definitively established. This is the optimal place to allocate bounds and rectangles.
4.  **`onLayout(changed, left, top, right, bottom)`:** The framework tells the View exactly where it is positioned on the screen relative to its parent. If the Custom View is a `ViewGroup`, it must now assign positions and sizes to all of its children.
5.  **`onDraw(Canvas)`:** The final step. The framework hands the View a `Canvas` object and says, "Render your visual representation here."

Mastering Custom Views requires mastering `onMeasure` and `onDraw`.

---

## 1. The Art of Negotiation: `onMeasure()`

Measurement in Android is a negotiation process between a parent `ViewGroup` (like a `LinearLayout` or `ConstraintLayout`) and your Custom View. The parent does not simply dictate a size; it passes down constraints known as `MeasureSpecs`.

A `MeasureSpec` is a 32-bit integer that cleverly packs two pieces of information: a **Mode** and a **Size**.

There are three possible Modes:
*   **`MeasureSpec.EXACTLY`:** The parent has determined an exact, rigid size for the view (e.g., the XML says `layout_width="100dp"` or `match_parent`). Your view *must* be this size.
*   **`MeasureSpec.AT_MOST`:** The parent has determined a maximum allowable size (e.g., the XML says `layout_width="wrap_content"`). Your view can be any size it wants, up to the specified limit.
*   **`MeasureSpec.UNSPECIFIED`:** The parent places no constraints on the view (rare, usually happens in scrolling views like `ScrollView`). Your view can be as large as it desires.

To build a robust Custom View, you must override `onMeasure` and properly interpret these specs to calculate your final dimensions. If you fail to override this properly, your `wrap_content` attributes will likely behave like `match_parent`.

### Implementing onMeasure

```kotlin
import android.content.Context
import android.util.AttributeSet
import android.view.View
import kotlin.math.min

class InteractiveGaugeView @JvmOverloads constructor(
    context: Context, attrs: AttributeSet? = null, defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {

    // The ideal size of this component if the parent places no restrictions on it
    private val desiredWidth = 400 // Usually calculated dynamically based on text or graphic size
    private val desiredHeight = 400

    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        
        // Extract the mode and size from the packed MeasureSpec integers
        val widthMode = MeasureSpec.getMode(widthMeasureSpec)
        val widthSize = MeasureSpec.getSize(widthMeasureSpec)
        
        val heightMode = MeasureSpec.getMode(heightMeasureSpec)
        val heightSize = MeasureSpec.getSize(heightMeasureSpec)

        // Calculate final width based on the mode
        val finalWidth = when (widthMode) {
            // Parent dictates exact size (match_parent or fixed dp)
            MeasureSpec.EXACTLY -> widthSize 
            
            // Parent gives a max limit (wrap_content). 
            // We take our desired size, but ensure we don't exceed the parent's limit.
            MeasureSpec.AT_MOST -> min(desiredWidth, widthSize) 
            
            // Parent doesn't care (ScrollView). Take our desired size.
            MeasureSpec.UNSPECIFIED -> desiredWidth
            
            // Fallback (should theoretically never happen)
            else -> desiredWidth 
        }

        // Calculate final height using the same logic
        val finalHeight = when (heightMode) {
            MeasureSpec.EXACTLY -> heightSize
            MeasureSpec.AT_MOST -> min(desiredHeight, heightSize)
            MeasureSpec.UNSPECIFIED -> desiredHeight
            else -> desiredHeight
        }

        // CRITICAL: You must call setMeasuredDimension. 
        // Failing to do so will result in an IllegalStateException crashing the app.
        setMeasuredDimension(finalWidth, finalHeight)
    }
}
```

By correctly implementing this negotiation, your Custom View will behave predictably regardless of the layout parameters a fellow developer assigns to it in the XML.

---

## 2. Rendering Pixels: `onDraw(Canvas)`

Once the view has been measured and laid out, the system invokes `onDraw()`. This is the core of your Custom View. The system provides you with a `Canvas` object. Think of the `Canvas` as the physical screen surface, and your tools are various methods like `drawRect()`, `drawCircle()`, `drawText()`, and `drawPath()`.

To draw anything on the canvas, you also need a `Paint` object. The `Paint` object dictates *how* the shape should be drawn (color, stroke width, text size, anti-aliasing, shadow effects).

### The Golden Rule of onDraw: Zero Allocations

Before diving into drawing logic, you must understand the absolute most critical rule of Custom View performance: **Never allocate objects (using the `new` keyword or calling object constructors) inside the `onDraw` method.**

Why? Because `onDraw` is highly volatile. If your view is animating, or if the user is scrolling the screen, `onDraw` might be called 60 or even 120 times per second to maintain a smooth framerate. 

If you instantiate a new `Paint` object or a new `RectF` object inside `onDraw`, you will be creating thousands of objects per second. This rapid object creation will rapidly fill the Java Heap memory. When the heap gets full, the Android Garbage Collector (GC) must pause the entire application to clean up the memory. These GC pauses cause visible stuttering, dropped frames, and a horrible user experience known as "jank."

You must pre-allocate all `Paint`, `Path`, and `Rect` objects as class-level variables or inside the constructor.

### Implementing onDraw Safely

Let's implement a simple, custom circular progress ring to demonstrate proper allocation and drawing techniques.

```kotlin
import android.content.Context
import android.graphics.Canvas
import android.graphics.Color
import android.graphics.Paint
import android.graphics.RectF
import android.util.AttributeSet
import android.view.View

class CircularProgressView @JvmOverloads constructor(
    context: Context, attrs: AttributeSet? = null, defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {

    // PRE-ALLOCATION: Define Paint objects at the class level
    private val backgroundPaint = Paint(Paint.ANTI_ALIAS_FLAG).apply {
        style = Paint.Style.STROKE
        strokeWidth = 20f
        color = Color.LTGRAY
        strokeCap = Paint.Cap.ROUND // Rounded edges
    }

    private val progressPaint = Paint(Paint.ANTI_ALIAS_FLAG).apply {
        style = Paint.Style.STROKE
        strokeWidth = 20f
        color = Color.parseColor("#6200EE") // Primary Purple
        strokeCap = Paint.Cap.ROUND
    }

    // PRE-ALLOCATION: Define RectF to hold bounds without allocating in onDraw
    private val boundsRect = RectF()
    
    // State variable
    private var progressPercentage = 65f

    // Update bounds only when the View's size actually changes
    override fun onSizeChanged(w: Int, h: Int, oldw: Int, oldh: Int) {
        super.onSizeChanged(w, h, oldw, oldh)
        
        // Calculate the drawing area, padding inward to account for the stroke width
        // so the stroke doesn't get clipped by the view boundaries.
        val padding = progressPaint.strokeWidth / 2f
        boundsRect.set(
            padding, 
            padding, 
            w.toFloat() - padding, 
            h.toFloat() - padding
        )
    }

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)

        // 1. Draw the background track (a full 360-degree circle)
        // Canvas drawing starts at 3 o'clock (0 degrees) and moves clockwise.
        canvas.drawArc(boundsRect, 0f, 360f, false, backgroundPaint)

        // 2. Draw the actual progress arc on top
        // We start at -90 degrees (12 o'clock) for a traditional progress bar look.
        val sweepAngle = (360f * (progressPercentage / 100f))
        canvas.drawArc(boundsRect, -90f, sweepAngle, false, progressPaint)
    }

    // Public method to allow Activity/Fragment to update the progress
    fun setProgress(progress: Float) {
        this.progressPercentage = progress
        // CRITICAL: invalidate() tells the Android framework that the visual state 
        // has changed and it needs to schedule a call to onDraw() as soon as possible.
        invalidate() 
    }
}
```

## Advanced Interaction: Handling Touch Events

A static drawing is visually pleasing, but a true Custom View should be interactive. To respond to user input, you must override the `onTouchEvent(MotionEvent)` method.

The `MotionEvent` object contains all the raw data about the user's interaction: the action type (Down, Move, Up, Cancel) and the X/Y coordinates of the touch.

```kotlin
    override fun onTouchEvent(event: MotionEvent): Boolean {
        // Extract the exact coordinate the user touched
        val touchX = event.x
        val touchY = event.y

        when (event.action) {
            MotionEvent.ACTION_DOWN -> {
                // The user just touched the screen.
                // Check if (touchX, touchY) intersects with your drawn shapes.
                
                // Return 'true' to tell the Android touch system: 
                // "I am handling this touch event. Send subsequent MOVE and UP events to me."
                return true 
            }
            MotionEvent.ACTION_MOVE -> {
                // The user is dragging their finger across the view.
                // You can update your state variables here (e.g., dragging a slider)
                // and call invalidate() to redraw the view at the new finger position.
            }
            MotionEvent.ACTION_UP -> {
                // The user lifted their finger. Finalize the interaction.
                performClick() // Good practice for accessibility support
            }
        }
        
        // If we don't care about the event, pass it back to the superclass
        return super.onTouchEvent(event)
    }
```

When building complex touch interactions involving scrolling or flinging, it is highly recommended to use helper classes like `GestureDetector` or `ViewDragHelper` rather than parsing raw `MotionEvent` mathematics yourself, as they handle the intricate edge cases of multi-touch and velocity calculation.

## Conclusion

Building Custom Views in Android is an essential skill that bridges the gap between standard app development and high-end graphical engineering. While the transition from declarative XML layouts to imperative Java/Kotlin Canvas drawing can feel daunting initially, mastering the `onMeasure` constraints and adhering strictly to the zero-allocation rule within `onDraw` will empower you to create highly performant, visually unique, and deeply interactive components that set your applications apart from the standard Material Design crowd.
