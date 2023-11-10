<script>
  import { onMount } from "svelte";
  let boxRef;
  export let style = {};
  let dragStyle = {
    ...style,
    transform: "translate(0, 0)",
  };
  let isDragging = false;
  let dragInfo = {
    offsetX: 0,
    offsetY: 0,
    endX: 0,
    endY: 0,
  };
  let distance = {
    t: 0,
    b: 0,
    l: 0,
    r: 0,
  };

  const formatStyle = (obj) => {
    if (typeof obj === "string") return obj;
    return Object.keys(obj)
      .reduce((style, key) => {
        const value = obj[key];
        if (value === null || value === undefined) {
          return style;
        }
        const unit =
          typeof value === "number" &&
          /width|height|top|right|bottom|left/i.test(key)
            ? "px"
            : "";
        return style.concat(
          `${key.replace(
            /([A-Z])/,
            (a) => `-${a.toLowerCase()}`
          )}:${value}${unit}`
        );
      }, [])
      .join(";");
  };

  const detectBoundary = () => {
    const style = boxRef.getBoundingClientRect();
    const { left, right, bottom, top } = style;
    const leftDistance = left;
    const rightDistance = window.innerWidth - right;
    const topDistance = top;
    const bottomDistance = window.innerHeight - bottom;
    const _distance = {
      t: topDistance,
      b: bottomDistance,
      l: leftDistance,
      r: rightDistance,
    };
    distance = _distance;
  };

  const setBoundary = (offsetX, offsetY) => {
    let newOffsetX = offsetX;
    let newOffsetY = offsetY;
    if (offsetX <= 0) {
      if (offsetX < -distance.l) {
        newOffsetX = -distance.l;
      } else {
        newOffsetX = offsetX;
      }
    } else {
      if (offsetX > distance.r) {
        newOffsetX = distance.r;
      } else {
        newOffsetX = offsetX;
      }
    }

    if (offsetY <= 0) {
      if (offsetY < -distance.t) {
        newOffsetY = -distance.t;
      } else {
        newOffsetY = offsetY;
      }
    } else {
      if (offsetY > distance.b) {
        newOffsetY = distance.b;
      } else {
        newOffsetY = offsetY;
      }
    }
    dragInfo.endX = newOffsetX;
    dragInfo.endY = newOffsetY;
    dragStyle.transform = `translate(${newOffsetX}px, ${newOffsetY}px)`;
  };

  const onTouchStart = (e) => {
    dragInfo.offsetX = (e.pageX || e.touches[0].pageX) - dragInfo.endX;
    dragInfo.offsetY = (e.pageY || e.touches[0].pageY) - dragInfo.endY;
    isDragging = true;
  };

  const onTouchEnd = () => {
    isDragging = false;
  };

  const onTouchMove = (e) => {
    if (isDragging) {
      const offsetX = e.touches[0].pageX - dragInfo.offsetX;
      const offsetY = e.touches[0].pageY - dragInfo.offsetY;
      setBoundary(offsetX, offsetY);
    }
  };

  const onMouseMove = (e) => {
    if (isDragging) {
      const offsetX = e.pageX - dragInfo.offsetX;
      const offsetY = e.pageY - dragInfo.offsetY;
      setBoundary(offsetX, offsetY);
    }
  };

  const onMouseUp = () => {
    document.removeEventListener("mousemove", onMouseMove);
    document.removeEventListener("mouseup", onMouseUp);
    isDragging = false;
  };

  const onMouseDown = (e) => {
    e.preventDefault();
    dragInfo.offsetX = e.pageX - dragInfo.endX;
    dragInfo.offsetY = e.pageY - dragInfo.endY;

    document.addEventListener("mousemove", onMouseMove);
    document.addEventListener("mouseup", onMouseUp);
    isDragging = true;
  };

  onMount(() => {
    detectBoundary();
  });
</script>

<div
  class="dragger"
  role="button"
  tabindex="0"
  style={formatStyle(dragStyle)}
  bind:this={boxRef}
  on:mousedown={onMouseDown}
  on:touchstart={onTouchStart}
  on:touchend={onTouchEnd}
  on:touchmove={onTouchMove}
>
  <slot />
</div>

<style>
  .dragger {
    position: fixed;
    z-index: 100;
    user-select: none;
    touch-action: none;
    cursor: pointer;
  }
</style>
