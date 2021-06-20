# 폐기된 노드 {#prunned-node}

폐기된(prunned) 비트코인 노드는 기술적으로는 여전히 비트코인 전체 노드이지만 디스크 공간을 절약하기 위해 로컬 저장소에서는 제거 됩니다. 스트라티스(stratis)의 비트코인 노드와 비트코인 코어는 모두 가지 치기 기능이 있습니다.

고전적인 full node와 비교하여, pruned node는 오래된 블록을 다른 노드에 제공할 수 없으며, 몇 개의 전체 노드 설정, 특히 비트코인 코어의 'txindex=1'을 사용할 수 없습니다.

가지치기를 고려하는 사람들은 전체 노드를 실행하는 다른 요구 사항도 낮추기를 원하는 경향이 있으므로, 비트코인 코어에 이 작업을 수행하는 방법에 대한 몇 가지 아이디어를 삽입하는 것이 좋습니다.


```
prune=550

maxconnections=8
listen=0
maxuploadtarget=144

checkblocks=1
checklevel=0

txindex=0
```
