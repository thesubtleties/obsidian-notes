## Box Model Fundamentals
```css
/* Box Model Components */
.box {
    content:    /* The actual content */
    padding:    /* Space between content and border */
    border:     /* The border around the padding */
    margin:     /* Space outside the border */
}
```

## Box Sizing Properties
```css
/* Default box sizing */
.box {
    box-sizing: content-box;    /* Width/height = content only */
}

/* Include padding and border in width/height */
.box {
    box-sizing: border-box;     /* Recommended for most cases */
}

/* Global box-sizing reset (place in CSS root) */
* {
    box-sizing: border-box;
}
```

## Margin Properties
```css
/* Individual sides */
.box {
    margin-top: 10px;
    margin-right: 20px;
    margin-bottom: 10px;
    margin-left: 20px;
}

/* Shorthand - clockwise from top */
.box {
    margin: 10px 20px 10px 20px;    /* top right bottom left */
    margin: 10px 20px;              /* top/bottom left/right */
    margin: 10px;                   /* all sides */
}

/* Center horizontally */
.box {
    margin: 0 auto;
}

/* Negative margins (pull element) */
.box {
    margin-left: -20px;
}
```

## Padding Properties
```css
/* Individual sides */
.box {
    padding-top: 10px;
    padding-right: 20px;
    padding-bottom: 10px;
    padding-left: 20px;
}

/* Shorthand - clockwise from top */
.box {
    padding: 10px 20px 10px 20px;   /* top right bottom left */
    padding: 10px 20px;             /* top/bottom left/right */
    padding: 10px;                  /* all sides */
}
```

## Border Properties
```css
/* Individual properties */
.box {
    border-width: 1px;
    border-style: solid;
    border-color: #000;
}

/* Shorthand */
.box {
    border: 1px solid #000;
}

/* Individual sides */
.box {
    border-top: 1px solid #000;
    border-right: 2px dashed #999;
    border-bottom: 3px dotted #666;
    border-left: 4px double #333;
}

/* Border radius */
.box {
    border-radius: 5px;                     /* all corners */
    border-radius: 5px 10px 15px 20px;      /* top-left, top-right, bottom-right, bottom-left */
    border-radius: 50%;                     /* circle (if width = height) */
}
```

## Width and Height
```css
/* Fixed dimensions */
.box {
    width: 200px;
    height: 100px;
}

/* Percentage-based */
.box {
    width: 50%;          /* relative to parent */
    height: 100vh;       /* viewport height */
}

/* Min and Max dimensions */
.box {
    min-width: 100px;
    max-width: 500px;
    min-height: 100px;
    max-height: 500px;
}
```

## Display Properties
```css
.box {
    display: block;              /* Full width, new line */
    display: inline;            /* Width of content, same line */
    display: inline-block;      /* Inline with block properties */
    display: flex;              /* Flexible box layout */
    display: grid;              /* Grid layout */
    display: none;              /* Hide element */
}
```

## Key Points & Gotchas
- `box-sizing: border-box` is generally preferred as it makes sizing more intuitive
- Margins collapse vertically between elements
- Percentage margins/paddings are calculated based on parent's width
- `auto` margins can only center elements horizontally, not vertically
- Inline elements ignore width/height properties
- Padding and border increase element size unless using `border-box`
- Negative margins can cause unexpected layouts - use with caution

## Best Practices
- Use `border-box` sizing for consistent layouts
- Prefer padding over margin for internal spacing
- Use relative units (%, em, rem) for responsive design
- Consider min/max dimensions for responsive layouts
- Use shorthand properties when setting multiple values
- Keep layout calculations simple by using `border-box`