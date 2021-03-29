### 炫彩小球

```javascript
class Ball {
  /**
   * @param parentNode: 父节点
   * @param camp: 存储所有小球的数组
   * @param params:
   **         x: x坐标
    **         y: y坐标
    **         r: 半径
    **         color = "default": 小球颜色。"default" / "random"
    */
  constructor(parentNode, camp, params) {
    if (!parentNode || !camp || typeof params !== "object") {
      throw new Error("请传递所有参数");
    }
    if (!(parentNode instanceof HTMLElement)) {
      throw new Error("parentNode必须是一个DOM节点");
    }
    const { x, y, r, color } = params;
    this.x = x;
    this.y = y;
    this.r = r;
    this.parentNode = parentNode;
    this.camp = camp;
    this.id = (Math.random() * Date.now()).toString(16);

    // 移动速度
    this.dx = this._randomSpeed();
    this.dy = this._randomSpeed();
    // 透明度
    this.opacity = 1;
    // 小球颜色
    if (color === "random") {
      this.color = this._randomColor();
    } else {
      this.color = "#fff";
    }
    // 小球节点
    this.dom = null;
  }

  init() {
    this.dom = document.createElement("div");
    this.parentNode.appendChild(this.dom);

    const { style } = this.dom;
    style.borderRadius = "50%";
    style.position = "absolute";
    style.backgroundColor = this.color;
    this._fresh();

    this.timer = setInterval(this.update.bind(this), 10);
  }

  update() {
    this.x += this.dx;
    this.y += this.dy;
    this.opacity -= 0.3 / this.r;
    this.r -= 0.1;
    if (this.r <= 0 || this.opacity <= 0) {
      clearInterval(this.timer);
      this.destroy();
    }

    this._fresh();
  }

  _fresh() {
    const { style } = this.dom;
    style.width = this.r + "px";
    style.height = this.r + "px";
    style.left = this.x + "px";
    style.top = this.y + "px";
    style.opacity = this.opacity;
  }

  destroy() {
    const idx = this.camp.indexOf(this);
    this.camp.splice(idx, 1);

    this.parentNode.removeChild(this.dom);
    delete this;
  }

  _randomInt(mi, ma) {
    return Math.floor(Math.random() * (ma - mi + 1)) + mi;
  }

  _randomFloat(mi, ma) {
    return Math.random() * (ma - mi) + mi;
  }

  _randomColor() {
    let ret = "#";
    for (let i = 0; i < 6; i++) {
      ret += this._randomInt(0, 16).toString(16);
    }
    return ret;
  }

  // 速度范围：(-1, 1)
  _randomSpeed() {
    const sign = Math.random() > 0.5 ? 1 : -1;
    return this._randomFloat(1, 2) * sign / 2;
  }
}

function throttle(fn, wait) {
  let timer = null;
  return function (args) {
    const that = this;
    if (timer === null) {
      timer = setTimeout(() => {
        fn.call(that, args);
        timer = null;
      }, wait);
    }
  }
}

const container = document.body;
const arr = [];

const BASE_RADIUS = 30;
const THROTTLE_DELAY = 40;
const COLOR_PROB = 0.03;

container.addEventListener("mousemove", throttle(function (e) {
  if (e.target !== container) return;
  const x = e.pageX;
  const y = e.pageY;
  const isInColor = Math.random() < COLOR_PROB;
  const r = isInColor ? BASE_RADIUS * 2 : BASE_RADIUS;
  const ball = new Ball(container, arr, {
    x: x - r / 2,
    y: y - r / 2,
    r,
    color: isInColor ? "random" : "default"
  });
  ball.init();
  arr.push(ball);
}, THROTTLE_DELAY), true);
```

