 ## orcal 
```sql
SELECT
        D1.yyyy,
        D1.yyyy || D2.MM AS yyyymm,
        D3.DEPT_ID,
        D3.DEPT_NAME,
        D3.RANKING
        FROM
        (
        SELECT
        (
        TO_NUMBER (TO_CHAR(SYSDATE, 'YYYY')) - 5 + LEVEL
        ) AS yyyy
        FROM
        DUAL CONNECT BY LEVEL  &lt;=  5
        ORDER BY
        yyyy DESC
        ) D1,
        (
        SELECT
        LPAD (LEVEL, 2, '0') mm
        FROM
        dual CONNECT BY LEVEL  &lt;= 12
        ) D2,
        S_DEPT D3
        WHERE
        D3.DEPT_PARENT_ID = 2

        AND D3.DEPT_TYPE != 256
```
## mysql 
```sql
SELECT
	TAB1.WEEK_NUM,
	TAB1.START_WEEKDAY,
	TAB1.END_WEEKDAY
FROM
	(
		SELECT
			@rownum := @rownum + 1 AS WEEK_NUM,
			detail.rownum AS START_WEEKDAY,
			detail.rownum1 AS END_WEEKDAY
		FROM
			(SELECT @rownum := 0) r,
			(
				SELECT
					DATE_FORMAT(
						@rownum2,
						'%Y-%m-%d %H:%i:%s'
					) AS rownum1,
					DATE_FORMAT(
						@rownum2 := DATE_SUB(@rownum2, INTERVAL 7 DAY),
						'%Y-%m-%d %H:%i:%s'
					) AS rownum
				FROM
					(
						SELECT
							@rownum2 := DATE_ADD(
								DATE_FORMAT(
									'2019-04-22 00:00:00',
									'%Y-%m-%d %H:%i:%s'
								),
								INTERVAL 7 DAY
							)
					) r,
					qzq_jqfx_calendar
				WHERE
					@rownum2 >= DATE_ADD(
						DATE_FORMAT(
							'2019-03-01',
							'%Y-%m-%d %H:%i:%s'
						),
						INTERVAL 7 DAY
					)
				AND @rownum2 <= DATE_ADD(
					DATE_FORMAT(
						'2019-04-22',
						'%Y-%m-%d %H:%i:%s'
					),
					INTERVAL 7 DAY
				)
			) AS detail
	) tab1,
	s_dept
WHERE
	DEL_FLAG = 0
AND DEPT_TYPE = 25002
```
mysql的需要一张表配合使用calendar
