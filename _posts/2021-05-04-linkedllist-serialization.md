---
layout: post
title:  "[LinkedList]: Serialization & Deserialization"
date:   2021-05-04 14:44:05 -0700
categories: programming leetcode
---
## 序列化/反序列化一个单链表
LeetCode上单链表题的输入都是形如`[1,2,3]`的字符串。于是我就想到了应该是用JSON的形式来进行序列化与反序列化的。`jackson`就可以派上用场了。首先来看如何使用：
{% highlight java %}
package leetcode.common.singlylinkedlist;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;

import static org.junit.jupiter.api.Assertions.*;

class ListNodeTest {

    @DisplayName("Common: SinglyLinkedList: Serialization and Deserialization")
    @ParameterizedTest(name = "serializeDeserialize({0}) = {0}")
    @CsvSource(value = { "[]", "[1]", "[1,2]", "[1,2,3]" }, delimiter = ';')
    void serializeDeserialize(final String input) {
        assertEquals(input, ListNode.toJson(ListNode.fromJson(input)));
    }
}
{% endhighlight %}

### 节点的定义
节点类需要做相应的处理：
{% highlight java %}
package leetcode.common.singlylinkedlist;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.annotation.JsonDeserialize;
import com.fasterxml.jackson.databind.annotation.JsonSerialize;

@JsonSerialize(using = ListNodeJsonSerializer.class)
@JsonDeserialize(using = ListNodeJsonDeserializer.class)
public class ListNode {
    public int val;
    public ListNode next;

    public ListNode(int val) {
        this.val = val;
    }

    public ListNode(int val, ListNode next) {
        this.val = val;
        this.next = next;
    }

    public static ListNode fromJson(String lst) {
        try {
            return lst == null ? null : new ObjectMapper().readValue(lst, ListNode.class);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
        return null;
    }

    public static String toJson(ListNode head) {
        try {
            return head == null ? "[]" : new ObjectMapper().writeValueAsString(head);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
        return null;
    }
}
{% endhighlight %}
### 序列化
{% highlight java %}
package leetcode.common.singlylinkedlist;

import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

public class ListNodeJsonSerializer extends JsonSerializer<ListNode> {

    @Override
    public void serialize(ListNode value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        gen.writeStartArray();
        if (value != null) {
            List<Integer> lst = traversal(value);
            for (Integer v : lst) {
                gen.writeNumber(v);
            }
        }
        gen.writeEndArray();
    }

    private List<Integer> traversal(ListNode head) {
        List<Integer> ret = new ArrayList<>();
        while (head != null) {
            ret.add(head.val);
            head = head.next;
        }
        return ret;
    }
}
{% endhighlight %}

### 反序列化
{% highlight java %}
package leetcode.common.singlylinkedlist;

import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.DeserializationContext;
import com.fasterxml.jackson.databind.JsonDeserializer;

import java.io.IOException;

public class ListNodeJsonDeserializer extends JsonDeserializer<ListNode> {
    @Override
    public ListNode deserialize(final JsonParser p, final DeserializationContext ctxt)
            throws IOException, JsonProcessingException {
        final String[] ss = p.getCodec().readValue(p, String[].class);
        ListNode ret = null;
        ListNode cur = null;
        for (final String s : ss) {
            final ListNode node = new ListNode(Integer.parseInt(s));
            if (ret == null) {
                ret = node;
            } else {
                cur.next = node;
            }
            cur = node;
        }
        return ret;
    }
}
{% endhighlight %}
