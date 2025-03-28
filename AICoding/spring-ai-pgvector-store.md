
# 插入表

表命名
```
PgVectorStore.builder(jdbcTemplate,embeddingModel).vectorTableName("VectorStore").build();
默认是：vector_store
```


表结构（或者说插入sql）
```
PgVectorStore# insertOrUpdateBatch

private void insertOrUpdateBatch(final List<Document> batch, final List<Document> documents, final List<float[]> embeddings) {  
String sql = "INSERT INTO " + this.getFullyQualifiedTableName() + " (id, content, metadata, embedding) VALUES (?, ?, ?::jsonb, ?) ON CONFLICT (id) DO UPDATE SET content = ? , metadata = ?::jsonb , embedding = ? ";  
this.jdbcTemplate.batchUpdate(sql, new BatchPreparedStatementSetter() {  
public void setValues(PreparedStatement ps, int i) throws SQLException {  
Document document = (Document)batch.get(i);  
String content = document.getText();  
String json = PgVectorStore.this.toJson(document.getMetadata());  
float[] embedding = (float[])embeddings.get(documents.indexOf(document));  
PGvector pGvector = new PGvector(embedding);  
StatementCreatorUtils.setParameterValue(ps, 1, Integer.MIN_VALUE, UUID.fromString(document.getId()));  
StatementCreatorUtils.setParameterValue(ps, 2, Integer.MIN_VALUE, content);  
StatementCreatorUtils.setParameterValue(ps, 3, Integer.MIN_VALUE, json);  
StatementCreatorUtils.setParameterValue(ps, 4, Integer.MIN_VALUE, pGvector);  
StatementCreatorUtils.setParameterValue(ps, 5, Integer.MIN_VALUE, content);  
StatementCreatorUtils.setParameterValue(ps, 6, Integer.MIN_VALUE, json);  
StatementCreatorUtils.setParameterValue(ps, 7, Integer.MIN_VALUE, pGvector);  
}  
  
public int getBatchSize() {  
return batch.size();  
}  
});  
}
```
