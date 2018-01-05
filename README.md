rvo2.js
========

#### Multi-agent collision avoidance library ####

A JavaScript port of UNC's [RVO2](http://gamma.cs.unc.edu/RVO2/) library.

As stated in the RVO2 documentation: local collision avoidance as provided by RVO2 Library should in principle be accompanied by global path planning that determines the preferred velocity of each agent in each time step of the simulation.

[Demo](http://paulwe.github.com/rvo2.js/examples/example1.html)

### Usage ###

Download the [minified library](http://paulwe.github.com/rvo2.js/lib/rvo2.min.js) and include it in your html.

```html
<script src="js/rvo2.min.js"></script>
```

This standard collision avoidance simulation places agents around a ring at antipodal positions from their goals.

```html
<canvas id="rvo-test" height="800" width="800" />
<script>
(function() {
  // parameter descriptions are given here: http://gamma.cs.unc.edu/RVO2/documentation/2.0/params.html
  var sim = new RVO.Simulator(2, 50, 5, 10, 10, 7, 1, [0, 0])
    , goals = []
    , center = [400, 400]
    , radius = 380
    , total = 100
    , timeStep = 13;

  // 添加 100 个小球, 每个小球都有不同的目标

  // 这里的数量由 total 定义, 通过数量计算角度 delta 和 position, 围成一圈
  for (var angle = Math.PI / total; angle < Math.PI * 2; angle += Math.PI / (.5 * total)) {
    var point = RVO.Vector.multiply([Math.sin(angle), Math.cos(angle)], radius);
    sim.addAgent(RVO.Vector.add(center, point));    // 添加 Agent
    goals.push(RVO.Vector.subtract(center, point)); // 保存对应的目标
  }

  var ivl = setInterval(function() {
    if (reachedGoals(sim, goals)) { // 如果所有小球都到达目标点
      clearInterval(ivl);           // 停止循环
    }
    else {                                  // 如果有小球没有到达目标点
      updateVisualization(sim);             // 刷新屏幕
      setPreferredVelocities(sim, goals);   // 给所有小球分配新的速度
      sim.doStep(); //走一步
    }
  }, timeStep);
})()

// checks to see if all agents have reached their goals
// 检查是不是所有小球都到达了目标
function reachedGoals(sim, goals) {
  for (var i = 0, len = sim.agents.length; i < len; i ++) {
    if (RVO.Vector.absSq(RVO.Vector.subtract(sim.agents[i].position, goals[i])) > 1) {
      return false;
    }
  }
  return true;
}

// assigns new velocities to all the agents
// 给所有小球分配新的速度
function setPreferredVelocities(sim, goals) {
  for (var i = 0, len = sim.agents.length; i < len; i ++) {
    // 嗯据当前目标重新获取目标方向向量
    var goalVector = RVO.Vector.subtract(goals[i], sim.agents[i].position);
    if (RVO.Vector.absSq(goalVector) > 1) {
      goalVector = RVO.Vector.normalize(goalVector);
    }
    sim.agents[i].prefVelocity = goalVector; // 更新
  }
}

// draws the agents
// Canvas 绘制
function updateVisualization(sim) {
  var ctx = document.getElementById('rvo-test').getContext('2d');
  ctx.clearRect(0, 0, 800, 800);
  ctx.fillStyle = 'rgba(200, 0, 0, .7)'

  for (var i = 0, len = sim.agents.length; i < len; i ++) {
    var agent = sim.agents[i];
    ctx.beginPath();
    ctx.arc(agent.position[0], agent.position[1], agent.radius * .9, 0, Math.PI * 2);
    ctx.closePath();
    ctx.fill();
  }
}
</script>
```
