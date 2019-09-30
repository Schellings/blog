---
title: Could not find parameter map
date: 2018-07-12 19:56:24
tags: 
    - Mybatis
---
修改前：
```yaml
<insert id="insertFile" parameterMap="link.guiderole.domain.GrFileVO">
        insert into gr_file
            (file_id, title,`path`,`format`,art_id,upload_date,upload_by,created_date,created_by,last_updated_date,last_updated_by,e_code)
        values
            (#{file_id}, #{title},#{path},#{format},#{artId},#{uploadTime},#{uploadBy},#{createdTime},#{createdBy},#{lastUpdatedTime},#{lastUpdatedBy},#{e_code})
</insert>
```

修改后：
```yaml
<insert id="insertFile" parameterType="link.guiderole.domain.GrFileVO">
        insert into gr_file
            (file_id, title,`path`,`format`,art_id,upload_date,upload_by,created_date,created_by,last_updated_date,last_updated_by,e_code)
        values
            (#{file_id}, #{title},#{path},#{format},#{artId},#{uploadTime},#{uploadBy},#{createdTime},#{createdBy},#{lastUpdatedTime},#{lastUpdatedBy},#{e_code})
</insert>
```

注意parameterMap与parameterType