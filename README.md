const VirtualizedListboxComponent = React.forwardRef(function VirtualizedListboxComponent(
  { children, style, overflowY = "auto", ...rest },
  ref
) {
  const itemData = React.Children.toArray(children);
  const ITEM_SIZE = 50;
  // Revert this to Math.min to fix the performance issue
  const HEIGHT = Math.min(8, itemData.length) * ITEM_SIZE;

  // We use outerElementType to merge MUI's props/styles with React-Window's container
  const OuterElement = React.useCallback(
    React.forwardRef((props, ref) => (
      <div
        ref={ref}
        {...props} // Props from FixedSizeList (includes style with correct height/width)
        {...rest}  // Props from MUI (role="listbox", onMouseDown, etc.)
        style={{
          ...props.style,
          ...style, // Merge MUI positioning styles
          // Your custom styling moved here:
          backgroundColor: theme.palette.secondary.main,
          borderRadius: "4px",
          boxShadow: "1px 1px 1px rgba(0, 0, 0, 0.2)",
          border: "1px solid #A2A8BD",
          maxHeight: "200px",
          overflowY,
          fontSize: "12px",
          position: "relative", // Ensure relative positioning for items
        }}
      />
    )),
    [style, rest, overflowY]
  );

  return (
    // No wrapper div here. We pass control directly to FixedSizeList.
    <FixedSizeList
      height={HEIGHT}
      width="100%"
      itemSize={ITEM_SIZE}
      itemCount={itemData.length}
      overscanCount={5}
      itemData={itemData}
      outerElementType={OuterElement}
      outerRef={ref} // CRITICAL: Gives MUI control over the actual scroll container
    >
      {({ index, style }) =>
        React.cloneElement(itemData[index], {
          style: {
            ...style,
            top: style.top,
            padding: "0 12px",
            // Fixes width issues inside virtual row
            width: "100%", 
            boxSizing: "border-box",
          },
        })
      }
    </FixedSizeList>
  );
});
