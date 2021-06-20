## 压缩列表
  使用目的：压缩空间，节省内存。
  
  使用场景：压缩列表是列表建和哈希建的底层实现之一。当一个列表键或者哈希键只包含少量列表项，而且列表项为小数值或者较短字符串的时候，redis就是使用压缩列表作为列表建或者哈希键的实现。
  
  内容结构组成以及作用：
  ![image](https://user-images.githubusercontent.com/37873889/122663675-fba08000-d1ce-11eb-946e-d99e14fa7f2b.png)
  
  ![image](https://user-images.githubusercontent.com/37873889/122663724-2b4f8800-d1cf-11eb-99ac-a623ce0f61bc.png)

## 列表节点
  相关结构：从上到下，从左到右分别记录了节点的前一个节点的长度占用的字节数，前一个节点的长度（这样就能结合当前的节点地址找到前一个节点的地址），节点自己的长度占用的字节数，节点自己的长度，节点的编码，节点内容。
  
  `typedef struct zlentry {
    unsigned int prevrawlensize, prevrawlen;
    unsigned int lensize, len;
    unsigned int headersize;
    unsigned char encoding;
    unsigned char *p;
  } zlentry`

  节点内容组成结构：每个节点内容都由previous_entry_length、encoding、content三部分组成，如图所示：
  ![image](https://user-images.githubusercontent.com/37873889/122663924-8a61cc80-d1d0-11eb-8e68-286c42636a8f.png)
  
  ![image](https://user-images.githubusercontent.com/37873889/122663981-0bb95f00-d1d1-11eb-9e66-33a1dcf5ceba.png)
  
  # 节点内容各部分
  previous_entry_length：该属性的占用的字节数是1或者5，当前一个节点的长度小于254时，该属性值只用一个字节存储；如果前一个节点长度大于或者等于254时，该属性的长度为5个字节，第一个字节记为0xfe，之后的四个字节存储长度值，而且是低内存存储长度值的高位字节。
  
  encoding：该属性存储了节点的content属性存储的内容的类型（字符创还是整数）和长度。该属性第一个字节的两个高位比特位用来表示编码类型。如图：
  ![image](https://user-images.githubusercontent.com/37873889/122664149-1fb19080-d1d2-11eb-8bea-559bb1458d84.png)

  content：该属性的内容有encoding属性决定是存储字符串还是整数。

## 连锁更新问题
  由于每个链表节点都由保存前一个链表节点的长度，而且会根据前一节点的长度使用不同长度的字节数存储，那就会存在一个问题。如果前一个节点的内容长度发生改变，而且长度从小于254到大于254，那么该节点存储的关于前节点的信息就会需要更新，而且需要使用更长的字节数去存储前节点的长度，这样就导致自己的长度也会增大，这样也就有可能使下一个节点进行更新，如此下去，知道最后一个节点。也就可能出现一个节点的内容发生变化，其后面的所有节点都需要一次进行更新，就出现了连锁更新现象，出现这个现象会导致效率变低。但是这个现象出现的几率很微小。
