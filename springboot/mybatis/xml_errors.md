# (Mybatis) List of **errors** when coding with `.xml` file 

## Formating error when using comaprisons `<`, `<=`, `>`, `>=`

Xml file interprets any **angle brackets** as **TAG**. When your code looks like:

```xml
	<select id="selectAllOrdersCopySeq" 
			resultType="com.ktnet.cbec.dataservice.domain.dao.OrdersCopySeqDao"
			parameterType="int">
		SELECT * 
		  FROM orders_copy_seq 
		 WHERE seq <= #{belowSeq}
		 ORDER BY user_id, time;
 	</select>
```

Following Error prints when Sringboot has run.

```
The content of elements must consist of well-formed character data or markup.
```

### FIX

Surround your angle bracket codes with `<![CDATA[content goes here]]>`.

```xml
	<select id="selectAllOrdersCopySeq" 
			resultType="com.ktnet.cbec.dataservice.domain.dao.OrdersCopySeqDao"
			parameterType="int">
		SELECT * 
		  FROM orders_copy_seq 
		 <![CDATA[WHERE seq <= #{belowSeq}]]>
		 ORDER BY user_id, time;
 	</select>
```
