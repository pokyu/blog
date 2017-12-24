---
title: Hadoop&GP&Oralce对比
date: 2017-09-25 22:07:09
tags: 
- Hadoop
- GreenPlum
- Oracle
categories: 
- Hadoop
- GreenPlum
- Oracle
---

- <font size=4 color=black><B>1、编写目的</B></font>  

	此次性能测试主要对比Hadoop，GP，Oracle的导入性能、查询性能。具体环境信息如下： 
 
	<font color=black><B>Hadoop节点：</B></font>

	<font color=black>  
	- 15台节点（12台数据节点，3台主节点），内存64G，CPU：E5-2640 2.0GHZ  
    - 290w左右，含270T硬盘，可用空间约为70T（三倍备份）
	</font> 

    <font color=black><B>GP信用卡分析库：</B><font color=black>
	- 16台节点，内存128G，CPU：E5-2690 3.0GHZ  
    - 500w左右，含270T硬盘，可用空间约为90T（两倍备份）
    </font>  
      
    <font color=black><B>Oracle信用卡分析库：</B></font><font color=black>
	- 2个节点  
    - 100w左右，磁盘空间另算
    </font>  

    <font color=red><B>注：测试均在环境较为宽松情况下进行，可能和平时结果稍有差别</B></font>   
- <font size=4 color=black><B>2、导入性能</B></font>  
	
	<font color=black><B>Hadoop导入性能：</B></font>  
	<table cellspacing="0" bordercolor="black">
	<tr>
		<td rowspan="3" align="center">文件原始大小</td>
		<td colspan="6" align="center" >Hadoop</td>
		<td colspan="2" align="center">GP</td>
		<td colspan="2" align="center">Oracle</td>
	</tr>
	<tr >
		<td colspan="2" align="center"  bgcolor="DCDCDC">未压缩</td>
		<td colspan="2" align="center" bgcolor="DCDCDC">Gzip</td>
		<td colspan="2" align="center" bgcolor="DCDCDC">Snappy</td>
		<td colspan="2" align="center" bgcolor="DCDCDC">五级压缩</td>
		<td colspan="2" align="center" bgcolor="DCDCDC">未压缩</td>
	</tr>
	<tr>
		<td width=40px >时间</td>
		<td width=40px>大小</td>
		<td width=40px>时间</td>
		<td width=40px>大小</td>
		<td width=40px>时间</td>
		<td width=40px>大小</td>
		<td width=40px>时间</td>
		<td width=40px>大小</td>
		<td width=40px>时间</td>
		<td width=40px>大小</td>
	</tr>
	<tr>
		<td bgcolor="DCDCDC">918M</td>
		<td bgcolor="DCDCDC">87s</td>
		<td bgcolor="DCDCDC">968.5M</td>
		<td bgcolor="DCDCDC">135s</td>
		<td bgcolor="DCDCDC">206M</td>
		<td bgcolor="DCDCDC">95s</td>
		<td bgcolor="DCDCDC">361M</td>
		<td bgcolor="DCDCDC"><font color="red"><B>11.8s</B></font></td>
		<td bgcolor="DCDCDC">289M</td>
		<td bgcolor="DCDCDC">76s</td>
		<td bgcolor="DCDCDC">1G</td>
	</tr>
	<tr>
		<td>14G</td>
		<td>258s</td>
		<td>13.9G</td>
		<td>275s</td>
		<td>1.7G</td>
		<td>290s</td>
		<td>3.3G</td>
		<td><font color="red"><B>157s</B></font></td>
		<td>4G</td>
		<td><font color="green"><B>1886s</B></font></td>
		<td>14G</td>
	</tr>
	<tr>
		<td bgcolor="DCDCDC">27G</td>
		<td bgcolor="DCDCDC">308s</td>
		<td bgcolor="DCDCDC">27G</td>
		<td bgcolor="DCDCDC">313s</td>
		<td bgcolor="DCDCDC">3.5G</td>
		<td bgcolor="DCDCDC"><font color="red"><B>270s</B></font></td>
		<td bgcolor="DCDCDC">7G</td>
		<td bgcolor="DCDCDC">337s</td>
		<td bgcolor="DCDCDC">7.3G</td>
		<td bgcolor="DCDCDC"><font color="green"><B>2135s</B></font></td>
		<td bgcolor="DCDCDC">27G</td>
	</tr>
	<tr>
		<td>40G</td>
		<td>421s</td>
		<td>44G</td>
		<td>421s</td>
		<td>13.4G</td>
		<td><font color="red"><B>350s</B></font></td>
		<td>22.4G</td>
		<td>452s</td>
		<td>25G</td>
		<td><font color="green"><B>3093s</B></font></td>
		<td>40G</td>
	</tr>
	</table>

	<font color=black><B>关于Hadoop压缩和非压缩：</B></font>  
	> <font color=black>压缩是在读进Hadoop后，在Hadoop内部完成。由于目前ETL和Hadoop之前是千兆网，读的速度可能和后端的处理速度差不多，所以对于大文件，效率是差不多的。如果是万兆网，非压缩效率比压缩效率快将近三分之一时间。
  
	<font color=black><B>关于Hadoop压缩格式：</B></font>  
	> <font color=black>GZIP压缩：  
	>> 优点：压缩比高，可达5-10倍；通用性高，可直接取到本地进行解压。  
	>> 缺点：每次压缩、解压只能起一个map进行处理，需要消耗较多资源以及时间。</font>   
	
	> <font color=black>SNAPPY压缩：  
	>> 优点：压缩、解压时间较快，消耗较少资源。  
	>> 缺点：压缩比低，只有2-3倍。通用性较低，无法在本地直接解压</font>    

	<font color=black><B>测试结果分析：</B></font>  
	> <font color=black>从数据上可看出，10G以上数据导入时，Hadoop优势开始显现。Hadoop可将大文件切分每一小块，并行处理。对于小文件（G级别以下）导入时，Hadoop处理较慢。因为每次导入均需一起个MR作业，需要消耗作业时间。</font>
  
- <font size=4 color=black><B>3、查询性能</B></font>  
	
	由于条件限制，此次测试仅在1G、 14G、27G及40G的表做一些查询测试，涉及count，where，group by，join等测试，具体如下：</font>  
	<table cellspacing="0" align=left>
	<tr>
		<td></td>
		<td bgcolor="DCDCDC">Hadoop未压缩</td>
		<td bgcolor="DCDCDC">HadoopGzip压缩</td>
		<td bgcolor="DCDCDC">HadoopSnappy压缩</td>
		<td bgcolor="DCDCDC">GP5级压缩</td>
		<td bgcolor="DCDCDC">Oracle未压缩</td>
	</tr>
	<tr>
		<td><B>1g count</B><br>select count(*) from ccm_base_trans;</td>
		<td width=100px>29s</td>
		<td>33s</td>
		<td>30s</td>
		<td>0.14s</td>
		<td>0.6s</td>
	</tr>
	<tr>
		<td bgcolor="DCDCDC"><B>14g count</B><br>select count(*) from ccm_expd_card_rmf_m_stat;</td>
		<td bgcolor="DCDCDC">40s</td>
		<td bgcolor="DCDCDC" >31s</td>
		<td bgcolor="DCDCDC">29s</td>
		<td bgcolor="DCDCDC">1.3s</td>
		<td bgcolor="DCDCDC">32s</td>
	</tr>
	<tr>
		<td><B>27g count</B><br>select count(*) from ccm_base_acct_inf_m;</td>
		<td>39s</td>
		<td>37s</td>
		<td>34s</td>
		<td>2.4s</td>
		<td>55s</td>
	</tr>
	<tr>
		<td bgcolor="DCDCDC"><B>40g count</B><br>select count(*) from ccm_base_application_info;</td>
		<td bgcolor="DCDCDC">60s</td>
		<td bgcolor="DCDCDC">52s</td>
		<td bgcolor="DCDCDC">48s</td>
		<td bgcolor="DCDCDC">5.3s</td>
		<td bgcolor="DCDCDC">199s</td>
	</tr>
	<tr>
		<td><B>1g sum</B><br>select max(txn_amt),min(txn_amt),avg(txn_amt)<br>from ccm_base_trans_1g;</td>
		<td>29s</td>
		<td>38s</td>
		<td>37s</td>
		<td>0.5s</td>
		<td>1s</td>
	</tr>
	<tr>
		<td bgcolor="DCDCDC"><B>14g sum</B><br>select max(txn_amt),min(txn_amt),avg(txn_amt)<br>from ccm_expd_card_rmf_m_stat_14g;</td>
		<td bgcolor="DCDCDC">39s</td>
		<td bgcolor="DCDCDC">40s</td>
		<td bgcolor="DCDCDC">38s</td>
		<td bgcolor="DCDCDC">1.5s</td>
		<td bgcolor="DCDCDC">34s</td>
	</tr>
	<tr>
		<td><B>27g sum</B><br>select<br>max(curr_bill_amt),min(curr_bill_amt),avg(curr_bill_amt)<br>from ccm_base_acct_inf_m_27g;</td>
		<td>46s</td>
		<td>59s</td>
		<td>47s</td>
		<td>2.5s</td>
		<td>58s</td>
	</tr>
	<tr>
		<td bgcolor="DCDCDC"><B>40g sum</B><br>select<br>max(monthly_mortgage_payment),<br>min(monthly_mortgage_payment),<br>avg(monthly_mortgage_payment)<br>from ccm_base_application_info_40g;</td>
		<td bgcolor="DCDCDC">50s</td>
		<td bgcolor="DCDCDC">73s</td>
		<td bgcolor="DCDCDC">60s</td>
		<td bgcolor="DCDCDC">5.4s</td>
		<td bgcolor="DCDCDC">159.25s</td>
	</tr>
	<tr>
		<td><B>1g group by</B><br>select<br>prim_card_becif_custid,max(txn_amt),min(txn_amt),avg(txn_amt)<br>from ccm_base_trans_1g<br>group by prim_card_becif_custid<br>order by prim_card_becif_custid limit 10;</td>
		<td>72s</td>
		<td>86s</td>
		<td>80s</td>
		<td>1.2s</td>
		<td>9.11s</td>
	</tr>
	<tr>
		<td bgcolor="DCDCDC"><B>14g group by</B><br>select<br>prim_card_becif_custid,max(txn_amt),min(txn_amt),avg(txn_amt)<br>from ccm_expd_card_rmf_m_stat_14g<br>group by prim_card_becif_custid<br>order by prim_card_becif_custid;</td>
		<td bgcolor="DCDCDC">130s</td>
		<td bgcolor="DCDCDC">150s</td>
		<td bgcolor="DCDCDC">130s</td>
		<td bgcolor="DCDCDC">2.7s</td>
		<td bgcolor="DCDCDC">87s</td>
	</tr>
	<tr>
		<td><B>27g group by</B><br>select<br>cycle_date,max(curr_bill_amt),min(curr_bill_amt),avg(curr_bill_amt)<br>from ccm_base_acct_inf_m_27g<br>group by cycle_date<br>order by cycle_date;</td>
		<td>70s</td>
		<td>75s</td>
		<td>69s</td>
		<td>3.1s</td>
		<td>87s</td>
	</tr>
	<tr>
		<td bgcolor="DCDCDC"><B>40g group by</B><br>select pre_service_years<br>from ccm_base_application_info_40g<br>group by pre_service_years<br>order by pre_service_years limit 10;</td>
		<td bgcolor="DCDCDC">76s</td>
		<td bgcolor="DCDCDC">95s</td>
		<td bgcolor="DCDCDC">85s</td>
		<td bgcolor="DCDCDC">5.8s</td>
		<td bgcolor="DCDCDC">100s</td>
	</tr>
	<tr>
		<td><B>14g join 27g</B><br>select<br>a.acct,sum(rec_3m_txn_sum),<br>sum(rec_3m_txn_cnt),sum(rec_3m_txn_amt)<br>from ccm_expd_card_rmf_m_stat_14g a<br>left outer join ccm_base_acct_inf_m_27g b on a.acct = b.acct<br>and b.org = '241'<br>group by a.acct;
		</td>
		<td>145s</td>
		<td>155s</td>
		<td>160s</td>
		<td>95s</td>
		<td>时间过长</td>
	</tr>
	<tr>
		<td bgcolor="DCDCDC"><B>27g join 40g</B><br>select count(*)<br>from ccm_base_acct_inf_m_27g a<br>left outer join ccm_base_application_info_40g b<br>on a.application_no = b.application_no<br>and a.becif_custid=b.becif_custid<br>where b.application_no is not null;</td>
		<td bgcolor="DCDCDC">100s</td>
		<td bgcolor="DCDCDC">130s</td>
		<td bgcolor="DCDCDC">120s</td>
		<td bgcolor="DCDCDC">15s</td>
		<td bgcolor="DCDCDC">412s</td>
	</tr>
	<tr>
		<td><B>3表关联 1个并发(40G，14G，27G)</B><br>select<br>T1.party_id,count(*),sum(T2.rec_3m_txn_sum),<br>sum(T2.rec_6m_txn_sum),sum(T3.last_petty_fee),sum(T4.txn_amt)<br>from ccm_base_application_info_test T1<br>left join ccm_expd_card_rmf_m_stat_test T2<br>on T1.party_id=T2.prim_card_party_id<br>left join ccm_base_acct_inf_m_test T3<br>on T1.party_id=T3.party_id</td>
		<td>297s</td>
		<td>300s</td>
		<td>280s</td>
		<td>17s</td>
		<td>565s</td>
	</tr>
	<tr>
		<td bgcolor="DCDCDC"><B>4表关联 1个并发(1G，14G，27G，40G)</B><br>select<br>T1.party_id,count(*),sum(T2.rec_3m_txn_sum),<br>sum(T2.rec_6m_txn_sum),sum(T3.last_petty_fee),sum(T4.txn_amt)<br>from ccm_base_application_info_test T1<br>left join ccm_expd_card_rmf_m_stat_test T2<br>on T1.party_id = T2.prim_card_party_id<br>left join ccm_base_acct_inf_m_test T3<br>on T1.party_id = T3.party_id<br>left join ccm_base_trans_test T4<br>on T1.party_id = T4.party_id<br>group by T1.party_id</td>
		<td bgcolor="DCDCDC">360s</td>
		<td bgcolor="DCDCDC">340s</td>
		<td bgcolor="DCDCDC">371s</td>
		<td bgcolor="DCDCDC">22s</td>
		<td bgcolor="DCDCDC">644s</td>
	</tr>
	</table>
	<font color=black><B>并行测试：（开多个窗口同时运行，取平均值）</B></font> 
	<table cellspacing="0">
	<tr>
		<td></td>
		<td bgcolor="DCDCDC">Hadoop未压缩</td>
		<td bgcolor="DCDCDC">HadoopGzip压缩</td>
		<td bgcolor="DCDCDC">HadoopSnappy压缩</td>
		<td bgcolor="DCDCDC">GP5级压缩</td>
		<td bgcolor="DCDCDC">Oracle未压缩</td>
	</tr>
	<tr>
		<td width=140px><B>1个并发count 45G</B></td>
		<td>62s</td>
		<td>57s</td>
		<td>40s</td>
		<td>12s</td>
		<td>45s</td>
	</tr>
	<tr>
		<td bgcolor="DCDCDC"><B>3个并发</B></td>
		<td bgcolor="DCDCDC">68s</td>
		<td bgcolor="DCDCDC">67s</td>
		<td bgcolor="DCDCDC">45s</td>
		<td bgcolor="DCDCDC">16s</td>
		<td bgcolor="DCDCDC">60s</td>
	</tr>
	<tr>
		<td><B>10个并发</B></td>
		<td>92s</td>
		<td>90s</td>
		<td>60s</td>
		<td>37s</td>
		<td>94s</td>
	</tr>
	<tr>
		<td bgcolor="DCDCDC"><B>20个并发</B></td>
		<td bgcolor="DCDCDC">124s</td>
		<td bgcolor="DCDCDC">114s</td>
		<td bgcolor="DCDCDC">75s</td>
		<td bgcolor="DCDCDC">62s</td>
		<td bgcolor="DCDCDC">207s</td>
	</tr>
	<tr>
		<td><B>30个并发</B></td>
		<td>200s</td>
		<td>180s</td>
		<td>120s</td>
		<td>110s</td>
		<td>410s</td>
	</tr>
	</table>
	<font color=black><B>测试结果分析：</B></font>  
	> <font color=black>从测试结果看，G级别数据内，GP的查询效率远高于Oracle和Hadoop。  
	> 对于10G以内数据，Oracle查询性能略高于Hadoop；10G以上数据，Hadoop性能开始慢慢优于Oracle。并且随着数据量的增长，Hadoop查询性能优势越来越明显，Oracle效率严重下降。</font>
  
	<font color=black><B>建议：</B></font>    
	> <font color=black>对于较大的表(10G以上)，并且涉及较复杂关联查询时，建议存放在Hadoop内。导入性能，查询性能均优于Oracle</font>
	