---
layout: post
comments: true
title:  "[Alg] Dijkstra's algorithm"
date:   2020-01-22
categories: Algorithm
tags: [Algorithm, Dijkstra's algorithm]
---

[Dijkstra's algorithm](https://brilliant.org/wiki/dijkstras-short-path-finder/)은 Greedy Search 이용하는 대표적인 algorithm으로서 shortest path를 구하는 데 주로 사용되는 algorithm 입니다. 각 Node와 Node를 연결하는 Edge의 가중치(weight)가 있고, bidirectional(양방향성) edge 이며 또한 Start Node에서 End Node까지의 가장 짧은 weight의 합을 가지는 path를 구하는 것이 목적(Shortest path)인 algorithm 입니다. 

Dijkstra's algorithm을 swift로 구현해 봤습니다. Dijkstra's algorithm은 여러가지 방법으로 구현이 가능하나 가장 효율적이라고 알려진 priority queue를 사용하는 방법으로 구현하였습니다. ([참고](https://medium.com/swiftly-swift/dijkstras-algorithm-in-swift-15dce3ed0e22))

![dijkstra image](https://d18l82el6cdm1i.cloudfront.net/uploads/X7rvS7Kbgc-dijkstra_animation.gif)
# Dive into the source code

## Vertex

Vertex = Node로 봐도 무방합니다. 여기서는 Graph의 각 꼭지점을 Vertex 라고 명기하고 구현하였습니다. 

>Vertex 구현시 struct을 써도 무방하나 (제가 그렇게 하려고 무던히 노력했습니다.) 각 vertex 및 edge, path를 parameter로 넘기는 경우가 있는데 struct는 아시다시피 value로 넘겨집니다. 즉 새로 생성이 되는 것이죠. 따라서 class로 구현하여 reference를 넘김으로서 기존 data를 유지하는 방향으로 구현하였습니다.

```swift
import UIKit
import Foundation

class Vertex {
    var key: String
    var edges: [Edge]
    var visited: Bool

    init(key: String, edges:[Edge] = [Edge](), visited:Bool = false) {
        self.key = key
        self.edges = edges
        self.visited = visited
    }
}
```
Vertex는 3가지 변수를 갖습니다.

1. key
2. edges
3. visited

Key는 해당 vertex가 가진 이름 입니다. Edges는 해당 vertex에서 뻗어있는 가지가 되겠습니다. 마지막으로 visited는 해당 vertex를 방문했는지 여부를 나타내 줍니다.

```swift
class Edge {
    var connectedTo: Vertex
    var weight: Int

    init(connectedTo: Vertex, weight: Int) {
        self.connectedTo = connectedTo
        self.weight = weight
    }
}
```

Edge는 2개의 변수를 갖고 있습니다.

1. connectedTo
2. weight

ConnectedTo는 해당 edge가 어느 vertex로 연결되어 있는지를 나타냅니다. 즉 이 edge를 따라가면 만날 수 있는 vertex를 갖고 있겠군요. Weight은 이 edge를 지나갈 때 내는 통행료? 비슷한 것으로 생각할 수 있습니다. 우리가 찾는 shortest path는 이 통행료들의 합이 가장 적은 path 이겠네요.

```swift
class Path: Hashable {

    let vertex: Vertex
    let totWeight: Int
    let previousPath: Path?

    init(vertex: Vertex, edge: Edge? = nil , path: Path? = nil) {
        if let previousPath = path, let edge = edge {
            self.totWeight = edge.weight + previousPath.totWeight
        } else {
            self.totWeight = 0
        }

        self.vertex = vertex
        self.previousPath = path
    }

    static func == (lhs: Path, rhs: Path) -> Bool {
        return lhs.vertex.key == rhs.vertex.key
    }

    func hash(into hasher: inout Hasher) {}
}
```
가장 중요한 path 입니다. Path는 현재 vertex까지 오기까지의 과정을 담은 class라 보시면 될 것 같습니다. 

사실 소스를 짜면서 가장 이해가 힘들었던 부분이기도 합니다. (제가 100% 짠 것이 아니라 이해하면서 짠 것이니 어려웠겠죠...ㅎ) 아무튼 좀 자세히 보겠습니다. 다음 3가지 변수를 제공합니다.

1. vertex
2. totWeight
3. previousPath

Vertex는 최근까지 search를 마친 vertex 입니다. TotWeight은 지금 vertex까지 오기까지의 weight의 합계 입니다. 마지막으로 previousPath는 여기 오기까지 바로 직전의 path 입니다. 이 previousPath 역시 previousPath를 변수로 갖고 있겠죠? 즉 previousPath는 Linked list 형태로 존재하겠네요.


```swift
func searchShortestPath(_ source: Vertex, _ destination: Vertex) -> Path? {

    // Priority Queue
    // 1
    var pathes: [Path] = [Path]() {
        didSet {
            pathes.sort { return $0.totWeight < $1.totWeight }
        }
    }

    // 2
    pathes.append(Path(vertex: source))

    // 3
    while (pathes.count > 0) {

        // 4
        let cheapestPath = pathes.removeFirst()

        // 5
        if cheapestPath.vertex.visited == true {
            continue
        }

        // 6
        if cheapestPath.vertex.key == destination.key {
            return cheapestPath
        }

        // 7
        cheapestPath.vertex.visited = true

        // 8
        for edge in cheapestPath.vertex.edges where edge.connectedTo.visited == false {

            // 9
            let newPath = Path(vertex: edge.connectedTo, edge: edge, path: cheapestPath)
            let oldPath = pathes.filter { $0.vertex.key == edge.connectedTo.key }.first
            
            // 10
            if let _oldPath = oldPath {
                if newPath.totWeight < _oldPath.totWeight{
                    pathes[pathes.firstIndex(of: _oldPath)!] = newPath
                }
            } else {
                pathes.append(newPath)
            }
        }
    }

    return nil
}
```
알고리즘을 한번 보겠습니다. 

1. 우선 pathes는 탐색할 path를 모아놓은 array 입니다. 하지만 그냥 array가 아닌, value가 set이 되면 array 내 path를 weight 기준으로 sorting하는 array 입니다. 즉 path 중 가장 거리가 짧은 path가 늘 앞에 위치하겠네요.
2. 시작 path는 1번 vertex부터 시작합니다. 당연히 시작이니 이전 path도 없을 것이고 edge도 없을 것 입니다. 현재 Vertex 1에 있으니 vertex 로서 source를 전달해 주며 path를 생성한 후 array에 append 합니다.
3. While loop를 돕니다. 언제까지? priority queue가 빌 때까지.
4. 우선 queue 맨 앞 element를 갖고 옵니다. 현재 가장 최 단거리 Path이며 처음 시작할 때는 당연히 source를 갖고 옵니다.
5. 방문했던 vertex인지 확인합니다. 이미 방문했다면 다음으로 짧은 거리의 path를 갖고 옵니다.
6. 만약 목적지에 도착했다면 while loop를 빠져 나옵니다. 여기서 이런 의문을 가진 분들이 있을 것 입니다. 작은 weight만 따라가다 목적지를 만났다고 바로 cheapest path라고 return 해 버리면 Greedy search의 맹점하고 비슷한 오류 아닌가요? 라고 말입니다. 아래의 경우 말입니다.

![Greedy search](https://d18l82el6cdm1i.cloudfront.net/uploads/xlck8z42EM-greedy-search-path-example.gif)

위의 그림에서 Greedy search는 Node 값이 큰 쪽으로 계속 search를 진행하게 됩니다. 결국에 자신이 생각하는 최적 해는 7+12+6이라 생각하지만 실제로는 7->3->99이거든요. 

***하지만***

생각을 잘 해보면 답이 어렵지 않게 나옵니다. 최종 목적지까지 간 cheapestPath가 위의 6번 로직에서 true를 return한다는 것은 이미 priority queue의 맨 앞에 있었다는 것 입니다. (5번에 의해 6번 로직을 탔겠죠.) 즉 목적지까지 도달했는데도 그 total weight 값이 priority queue 맨 앞에 위치한다면, 당연히 그 path는 shortest가 분명합니다. 반대로 만약 목적지까지 도달한 path의 길이가 priority queue의 맨 앞에 위치하지 않는다면? 즉 목적지까지 도달하기 위한 total weight보다 적은 total weight이 현재 있다면, 그 path 를 5번 에서 갖고와서 다시 loop를 돌 것입니다. 괜히 걱정했네요? 

7. 반문한 vertex는 visit = true로 set 해줍니다.
8. 이제 현재 path의 가지(edge)들에 대해서 또 다시 탐색을 시작해야 합니다. for loop을 통해 현재 방문하지 않은 vertex에 대한 path를 만듭니다. 
9. 새로 만들 path(new) 그리고 이미 만들어져 있는 path)(old를 변수에 저장합니다.
10. 이미 저장되어 있는 path가 있으면, 그 total weight이 새로 만들어진 path의 total weight과 비교하여 좀 더 적은 쪽으로 path를 바꿔치기 합니다. 이미 저장되어 있는 path가 없으면 pathes에 append하고 다시 loop를 돕니다.

```swift
let hongkong = Vertex.init(key: "HongKong")
let sidney = Vertex.init(key: "Sidney")
let seoul = Vertex.init(key: "Seoul")
let tyoko = Vertex.init(key: "Tyoko")
let washinton = Vertex.init(key: "Washinton")
let mexico = Vertex.init(key: "Mexico")

hongkong.edges.append(Edge.init(connectedTo: seoul, weight: 14))
hongkong.edges.append(Edge.init(connectedTo: sidney, weight: 9))
hongkong.edges.append(Edge.init(connectedTo: tyoko, weight: 7))
tyoko.edges.append(Edge.init(connectedTo: sidney, weight: 10))
sidney.edges.append(Edge.init(connectedTo: seoul, weight: 2))
tyoko.edges.append(Edge.init(connectedTo: washinton, weight: 15))
sidney.edges.append(Edge.init(connectedTo: washinton, weight: 9))
washinton.edges.append(Edge.init(connectedTo: mexico, weight: 6))
seoul.edges.append(Edge.init(connectedTo: mexico, weight: 9))

let source = hongkong
let destination = mexico

// 1
if let path = searchShortestPath(source, destination) {
    var intermediatePath = path
    var arr: [Path] = []
    while let _path = intermediatePath.previousPath {
        arr.append(_path)
        intermediatePath = _path
    }

    print("Found shortest path \(arr.reversed().map { $0.vertex.key}) -> \(destination.key)")

} else {
    print("Can't find path")
}
```

맨 위 그림처럼 graph를 만들고 알고리즘을 돌렸습니다. SearchShortestPath의 return 값인 path에서 previousPath를 loop 돌면서 Shortest path를 구할 수 있습니다. 



