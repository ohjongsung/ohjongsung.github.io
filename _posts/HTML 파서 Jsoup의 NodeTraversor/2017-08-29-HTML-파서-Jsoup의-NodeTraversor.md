보통 크롤러를 만들때, 해당 HTML 구조를 파악하고 NavigatingAPI를 사용해서 필요한 DOM을 선택하고 값을 추출하거나 작업을 했다. 그런데 body 태그 안의 모든 DOM 중에서 조건에 맞는 DOM이면 특정 작업을 해야 할때는 어떻게 해야 할까? 특별한 규칙성이 없다면 모든 DOM을 순회하면서 일일이 조건에 맞는지 확인해야 한다. 그래서 HTML DOM은 트리 구조라고 할 수 있으니, 트리 순회 알고리즘을 적용하면 되지 않을까 생각했다. 그런데 내가 생각할 수준이면 이미 만들어져 있지 않을까 했는데, 역시나 [API](https://jsoup.org/apidocs/)를 뒤져보니 나온다. 그것이 org.jsoup.select 패키지의 NodeTraversor이다.

## NodeTraversor의 traverse 메소드

```java
    public void traverse(Node root) {
        Node node = root;
        int depth = 0;
        
        while (node != null) {
            visitor.head(node, depth);
            if (node.childNodeSize() > 0) {
                node = node.childNode(0);
                depth++;
            } else {
                while (node.nextSibling() == null && depth > 0) {
                    visitor.tail(node, depth);
                    node = node.parentNode();
                    depth--;
                }
                visitor.tail(node, depth);
                if (node == root)
                    break;
                node = node.nextSibling();
            }
        }
    }
```

코드를 보면 재귀호출 사용해서 순회하지 않는다. 아무래도 HTML DOM은 이진 트리가 아닌데다가 레벨도 제각각일테니 StackOverflowException 발생 가능성을 생각하면 당연한듯 하다. while 문의 코드를 보면, visitor.head 메소드와 visitor.tail 메소드가 있다. 이 visitor는 NodeVisitor 인터페이스의 구현체로 사용자가 head 메소드와 tail 메소드에 원하는 작업을 구현해서 NodeTraversor 객체 생성 시, 주입하면 된다. 이 두 메소드가 각각 preOrder (전위 순회), postOrder(후위 순회)를 하며 실행된다.

## NodeTraversor 사용 예제

```java
    Document doc = Jsoup.parse(html);

    NodeTraversor nodeTraversor = new NodeTraversor(new NodeVisitor() {
        @Override 
        public void head(Node node, int depth) {
            // 전위 순회
        }

        @Override 
        public void tail(Node node, int depth) {
            // 후위 순회
            if (node instanceof TextNode) { // 특정 조건
                // 기능 구현...또는 작업이 적용되어야 할 노드 수집
            }
        }
    });
    nodeTraversor.traverse(doc.body());
```

이 정도면 어떻게 사용해야 할지 감이 잡힐 것이다. preOrder (전위 순회)를 적용해서 작업하려면 head 메소드를 구현하면 되고, postOrder(후위 순회)를 적용하려면 tail 메소드를 구현하면 된다.

## 마무리

이런 코드를 보다 보니, 자료구조와 알고리즘 공부를 열심히 해야겠다는 생각이 든다. 조금이라도 공부를 했으니 만들어 볼 생각을 했고, API를 뒤져서 찾아낼 수도 있었으니까.

## Reference
* <https://jsoup.org/apidocs/>
* <http://edoli.tistory.com/95>