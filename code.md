```java
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.github.pagehelper.PageInfo;
import com.google.common.collect.Maps;
import net.sourceforge.jtds.jdbc.ClobImpl;
import org.apache.commons.lang.StringUtils;
import org.apache.http.HttpEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.conn.ssl.SSLConnectionSocketFactory;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.protocol.HTTP;
import org.apache.http.ssl.SSLContexts;
import org.apache.http.util.Args;
import org.apache.http.util.CharArrayBuffer;
import org.apache.http.util.EntityUtils;
import org.springframework.stereotype.Service;
import org.springframework.util.Assert;

import javax.annotation.Resource;
import javax.net.ssl.SSLContext;
import javax.servlet.http.HttpServletRequest;
import java.io.*;
import java.math.BigDecimal;
import java.net.InetAddress;
import java.net.UnknownHostException;
import java.nio.charset.Charset;
import java.security.KeyStore;
import java.sql.Clob;
import java.sql.SQLException;
import java.util.*;
import java.util.stream.Collectors;

@Service
public class RedPaperServiceImpl {

    /**
     * 签名Key
     */
    private static String KEY = Config.getString("config.wx.sign.key");

    /**
     * APP_ID
     */
    private static String APP_ID = Config.getString("config.wx.app.id");

    /**
     * 微信发红包接口url
     */
    private static String RED_PAPER_URL = "https://api.mch.weixin.qq.com/mmpaymkttransfers/sendredpack";

    /**
     * 商户ID(证书密码)
     */
    private static String CERT_VALUE = Config.getString("config.wx.cert.value");

    /**
     * 证书绝对路径
     */
    private static String CERT_PATH = Config.getString("config.wx.cert.path");

    /**
     * 发红包者名称（）
     */
    private static String sendName = "";

    /**
     * 祝福语()
     */
    private static String wishing = "";

    /**
     * 活动名称()
     */
    private static String actName = "";

    /**
     * 备注（感谢您的参与）
     */
    private static String remark = "感谢您的参与";

    /**
     * MattersDAO
     */
    @Resource
    private MattersDAO mattersDAO;

    /**
     * HttpServletRequest
     */
    @Resource
    private HttpServletRequest request;

    /**
     * 红包Service
     */
    @Resource
    private RedPaperDAO redPaperDAO;

    /**
     * 通用Dao
     */
    @Resource
    private CommonDAO commonDAO;

    /**
     * Mapper初始化
     *
     * @return
     */
    @Override
    protected BaseDAO<Matters> getDAO() {
        return mattersDAO;
    }

    @Override
    public void insertMatters(Matters matters) {
        mattersDAO.insertMatters(matters);
    }

    @Override
    public List<Matters> selectAll() {
        Matters matters = new Matters();
        matters.setDelFlag(false);
        return mattersDAO.select(matters);
    }

    @Override
    public List<Matters> buildMatters(Long parentId) {
        final List<Matters> mattersList = selectAll();
        List<Matters> lists = mattersList.stream().sorted((m1, m2) -> Ids.compare(m1.getRanking(), m2.getRanking()))
                .filter(p -> Ids.eq(parentId, p.getMattersTypeId())).collect(Collectors.toList());
        return buildTree(lists, mattersList);
    }

    /**
     * 分页获取事项列表
     *
     * @param param
     * @return
     */
    @Override
    public PageInfo<Matters> selectPageByParam(Matters param) {
        return mattersDAO.selectPageByParam(param);
    }

    /**
     * 根据细类Id删除细类
     *
     * @param mattersIds
     */
    @Override
    public void deleteMattersByIds(Long[] mattersIds) {
        for (Long l : mattersIds) {
            Matters matters = new Matters();
            matters.setDelFlag(true);
            matters.setMattersId(l);
            mattersDAO.updateByParam(matters);
        }
    }

    /**
     * 保存事项细类
     *
     * @param param
     */
    @Override
    public void saveMatters(Matters param) {
        Assert.notNull(param, "param not be null");

        if (Ids.verifyId(param.getMattersId())) {
            mattersDAO.updateByPrimaryKeySelective(param);
        } else {
            Assert.notNull(param.getHasExistChild(), "hasExistChild not be null");
            param.setDelFlag(false);
            param.setCreateTime(new Date());
            param.setCreateUid(LoginContext.getUserId());
            mattersDAO.insertSelective(param);
        }
    }

    /**
     * 细类升序
     *
     * @param otherObj
     * @return
     */
    @Override
    public List<Matters> getMaxMenuLevel(Matters otherObj) {
        return mattersDAO.getMaxMenuLevel(otherObj);
    }

    /**
     * 降序细类
     *
     * @param otherObj
     * @return
     */
    @Override
    public List<Matters> getMinMenuLevel(Matters otherObj) {
        return mattersDAO.getMinMenuLevel(otherObj);
    }

    public List<Matters> buildTree(List<Matters> list, List<Matters> paramList) {
        for (Matters matters : list) {
            if (matters.getHasExistChild() == 1) {
                List<Matters> list1 = paramList.stream().sorted((m1, m2) -> Ids.compare(m1.getRanking(), m2.getRanking()))
                        .filter(p -> Ids.eq(p.getMattersParentId(), matters.getMattersId()))
                        .collect(Collectors.toList());
                matters.setMattersList(buildTree(list1, paramList));
            }
        }
        return list;
    }

    /**
     * 通道数据插入红包表
     *
     * @param inviteCode
     * @return
     */
    public RedPaper transResultInToRedPaper(String inviteCode) {
        RedPaper redPaper =null;
        //通过兑换码获取数据
        String tableName = "o_open_result_in";
        Map<String, Object> conditionMap = Maps.newHashMap();
        conditionMap.put("oc_uuid", inviteCode);
        conditionMap.put("result_status", 0);
        conditionMap.put("del_flag", 0);
        Integer limit = 1;
        //根据验证码和result_status从外网sqlServer获取唯一条记录
        List<HashMap> mapList = commonDAO.selectByCondition(DataSourceEnum.SqlServer, tableName, conditionMap, limit);

        if (mapList.size() != 0) {

            Clob j = (Clob) (mapList.get(0).get("text"));
            String reString = "";
            Reader is = null;// 得到流
            try {
                is = j.getCharacterStream();
            } catch (SQLException e) {
                logger.error("异常",e);
            }
            BufferedReader br = new BufferedReader(is);
            String s = null;
            try {
                s = br.readLine();
            } catch (IOException e) {
                logger.error("io异常",e);
            }
            StringBuffer sb = new StringBuffer();
            while (s != null) {// 执行循环将字符串全部取出付值给StringBuffer由StringBuffer转成STRING
                sb.append(s);
                try {
                    s = br.readLine();
                } catch (IOException e) {
                    logger.error("io异常",e);
                }
            }
            reString = sb.toString();
            redPaper = JSON.parseObject(reString,RedPaper.class);
            int insertResult = redPaperDAO.insertSelective(redPaper);
            if (insertResult == 1) {
                //如果插入到红包表成功，那么将通道这条数据忽略掉
                Map<String, Object> updateConditionMap = Maps.newHashMap();
                updateConditionMap.put("result_status", 1);
                updateConditionMap.put("del_flag", 1);
                commonDAO.updateByCondition(DataSourceEnum.SqlServer, tableName, updateConditionMap, conditionMap);
            }
        }
        return redPaper;
    }

    /**
     * 发送红包
     *
     * @param openId
     * @param money
     * @return
     */
    public String sendRedPaper(String openId, Integer money) {

        //商户订单号
        String orderNNo = System.currentTimeMillis() + "";
        //红包参数
        Map<String, Object> paramMap = getParamMap(orderNNo, openId, money, RedPacketConstant.actName, RedPacketConstant.wishing, RedPacketConstant.remark, RedPacketConstant.sendName);
        //发送的报文参数
        String xml = createXML(paramMap);
        try {
            String resultXml = doSend(RedPacketConstant.RED_PAPER_URL, xml);
            //红包发送成功，状态判断
            if (resultXml.indexOf("<result_code><![CDATA[SUCCESS]]></result_code>") > -1) {
                return "1";
            } else if (resultXml.indexOf("NO_AUTH") > -1) {
                //发放失败，此请求可能存在风险，已被微信拦截
                return RedPacketConstant.NO_AUTH;
            } else if (resultXml.indexOf("SENDNUM_LIMIT") > -1) {
                //该用户今日领取红包个数超过限制
                return RedPacketConstant.SENDNUM_LIMIT;
            } else if (resultXml.indexOf("MONEY_LIMIT") > -1) {
                //红包金额发放限制
                return RedPacketConstant.MONEY_LIMIT;
            } else if (resultXml.indexOf("SEND_FAILED") > -1) {
                //红包发放失败,请更换单号再重试
                return RedPacketConstant.SEND_FAILED;
            } else if (resultXml.indexOf("SYSTEMERROR") > -1) {
                //请求已受理，请稍后使用原单号查询发放结果
                return RedPacketConstant.SYSTEMERROR;
            } else if (resultXml.indexOf("NOTENOUGH") > -1) {
                //帐号余额不足，请到商户平台充值后再重试
                return RedPacketConstant.NOTENOUGH;
            } else {
                //其它错误
                return RedPacketConstant.OTHER;
            }
        } catch (Exception e) {
            logger.error("接口发送失败", e);
        }
        return "0";
    }

    /**
     * 发送红包
     *
     * @param openid
     * @param inviteCode
     * @return
     */
    @Override
    public String send(String openid, String inviteCode) {

        //将通道（o_open_result_in表）数据插入红包表，并忽略o_open_result_in的数据
        RedPaper redPaper =  transResultInToRedPaper(inviteCode);
        //如果成功
        String result = null;
        String packetResult = null;
        if(redPaper == null){
            return "";
        }
        String check = checkAESData(redPaper);
        if("1".equals(check)){
            if (redPaper != null) {
                //待发红包金额
                String money = redPaper.getRedPaper();
                //元转成分
                Integer reallyMoney = Integer.valueOf(changeY2F(String.valueOf(money)));
                //发红包
                if (reallyMoney > 20000.00) {
                    int max = reallyMoney / 20000 + (reallyMoney % 20000 > 0 ? 1 : 0);
                    for (int i = 1; i <= max; i++) {
                        int m = reallyMoney - 20000 >= 0 ? 20000 : reallyMoney;
                        if (i != max) {
                            packetResult = sendRedPaper(openid, m);
                            if ("1".equals(packetResult)) {
                                reallyMoney -= m;
                                // 没有发送完，不更改已经领取状态，但是要更新钱的数量
                                redPaper.setRedPaper(changeF2Y(reallyMoney.toString()));
                            } else {
                                // 发送失败将余额存入留待下次发送
                                redPaper.setRedPaper(changeF2Y(reallyMoney.toString()));
                                // 将状态改为发送失败
                                redPaper.setResultStatus(RedPacketConstant.SENT_FAIL);
                                redPaper.setRedPacketReason(packetResult);
                                // 推送到
                                pushRedPaper(redPaper);
                            }
                        } else {
                            //到最后一次，更改发送状态
                            packetResult = sendRedPaper(openid, m);
                            if ("1".equals(packetResult)) {
                                reallyMoney -= m;
                                // 将状态改为发送成功
                                redPaper.setResultStatus(RedPacketConstant.SENT_SUCCESS);
                            } else {
                                // 发送失败将余额存入留待下次发送
                                redPaper.setRedPaper(changeF2Y(reallyMoney.toString()));
                                // 将状态改为发送失败
                                redPaper.setResultStatus(RedPacketConstant.SENT_FAIL);
                                redPaper.setRedPacketReason(packetResult);
                            }
                            // 推送到
                            pushRedPaper(redPaper);
                        }
                        redPaperDAO.updateByPrimaryKeySelective(redPaper);
                    }
                } else {
                    //小于200元直接发送成功后，直接更改已经领取状态
                    packetResult = sendRedPaper(openid, reallyMoney);
                    if ("1".equals(packetResult)) {
                        // 将状态改为发送成功
                        redPaper.setResultStatus(RedPacketConstant.SENT_SUCCESS);
                    } else {
                        // 将状态改为发送失败
                        redPaper.setResultStatus(RedPacketConstant.SENT_FAIL);
                        redPaper.setRedPacketReason(packetResult);
                    }
                    // 推送到
                    pushRedPaper(redPaper);
                    redPaperDAO.updateByPrimaryKeySelective(redPaper);
                }
            } else {
                result = "error";
            }
        }else {
            // 将状态改为发送失败
            redPaper.setResultStatus(RedPacketConstant.SENT_FAIL);
            redPaper.setRedPacketReason(RedPacketConstant.ILLEGAL);
            // 推送到内
            pushRedPaper(redPaper);
            redPaperDAO.updateByPrimaryKeySelective(redPaper);
        }

        return result;
    }

    /**
     * 将分为单位的转换为元 （除100）
     *
     * @param amount
     * @return
     * @throws Exception
     */
    public static String changeF2Y(String amount) {
        if (!amount.matches("\\-?[0-9]+")) {
            try {
                throw new Exception("金额格式有误");
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return BigDecimal.valueOf(Long.valueOf(amount)).divide(new BigDecimal(100)).toString();
    }

    /**
     * 金额转换
     *
     * @param amount
     * @return
     */
    public static String changeY2F(String amount) {
        String currency = amount.replaceAll("\\$|\\￥|\\,", "");  //处理包含, ￥ 或者$的金额
        int index = currency.indexOf(".");
        int length = currency.length();
        Long amLong = 0l;
        if (index == -1) {
            amLong = Long.valueOf(currency + "00");
        } else if (length - index >= 3) {
            amLong = Long.valueOf((currency.substring(0, index + 3)).replace(".", ""));
        } else if (length - index == 2) {
            amLong = Long.valueOf((currency.substring(0, index + 2)).replace(".", "") + 0);
        } else {
            amLong = Long.valueOf((currency.substring(0, index + 1)).replace(".", "") + "00");
        }
        return amLong.toString();
    }

    /**
     * 开始发送
     *
     * @param url
     * @param data
     * @return
     * @throws Exception
     */
    public String doSend(String url, String data) throws Exception {

        KeyStore keyStore = KeyStore.getInstance("PKCS12");
        FileInputStream instream = new FileInputStream(new File(CERT_PATH));//证书绝对路径
        try {
            keyStore.load(instream, CERT_VALUE.toCharArray());//商户号
        } finally {
            instream.close();
        }
        SSLContext sslcontext = SSLContexts.custom()
                .loadKeyMaterial(keyStore, CERT_VALUE.toCharArray())//商户号
                .build();
        SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslcontext, new String[]{"TLSv1"}, null,
                SSLConnectionSocketFactory.BROWSER_COMPATIBLE_HOSTNAME_VERIFIER);
        CloseableHttpClient httpclient = HttpClients.custom().setSSLSocketFactory(sslsf).build();
        try {
            HttpPost httpost = new HttpPost(url);  // 设置响应头信息
            httpost.addHeader("Connection", "keep-alive");
            httpost.addHeader("Accept", "*/*");
            httpost.addHeader("Content-Type", "application/x-www-form-urlencoded; charset=UTF-8");
            httpost.addHeader("Host", "api.mch.weixin.qq.com");
            httpost.addHeader("X-Requested-With", "XMLHttpRequest");
            httpost.addHeader("Cache-Control", "max-age=0");
            httpost.addHeader("User-Agent", "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.0) ");
            httpost.setEntity(new StringEntity(data, "UTF-8"));
            CloseableHttpResponse response = httpclient.execute(httpost);
            try {
                HttpEntity entity = response.getEntity();
                //获取一个utf-8编码的串
                String jsonStr = toStringInfo(response.getEntity(), "UTF-8");
                EntityUtils.consume(entity);
                return jsonStr;
            } finally {
                response.close();
            }
        } finally {
            httpclient.close();
        }
    }

    /**
     * 转换为一个串（UTF-8格式）
     *
     * @param entity
     * @param defaultCharset
     * @return
     * @throws Exception
     * @throws IOException
     */
    private String toStringInfo(HttpEntity entity, String defaultCharset) throws Exception, IOException {
        final InputStream instream = entity.getContent();
        if (instream == null) {
            return null;
        }
        try {
            Args.check(entity.getContentLength() <= Integer.MAX_VALUE,
                    "HTTP entity too large to be buffered in memory");
            int i = (int) entity.getContentLength();
            if (i < 0) {
                i = 4096;
            }
            Charset charset = null;

            if (charset == null) {
                charset = Charset.forName(defaultCharset);
            }
            if (charset == null) {
                charset = HTTP.DEF_CONTENT_CHARSET;
            }
            final Reader reader = new InputStreamReader(instream, charset);
            final CharArrayBuffer buffer = new CharArrayBuffer(i);
            final char[] tmp = new char[1024];
            int l;
            while ((l = reader.read(tmp)) != -1) {
                buffer.append(tmp, 0, l);
            }
            return buffer.toString();
        } finally {
            instream.close();
        }
    }

    /**
     * 拼接红包发送所带xml格式参数
     *
     * @param map
     * @return
     */
    private String createXML(Map<String, Object> map) {
        String xml = "<?xml version=\"1.0\" encoding=\"utf-8\"?><xml>";
        Set<String> set = map.keySet();
        Iterator<String> i = set.iterator();
        while (i.hasNext()) {
            String str = i.next();
            xml += "<" + str + ">" + "<![CDATA[" + map.get(str) + "]]>" + "</" + str + ">";
        }
        xml += "</xml>";
        return xml;
    }

    /**
     * 获取签名，并创建xml格式参数
     *
     * @param orderNNo
     * @param openId
     * @param money
     * @param actName
     * @param wishing
     * @param remark
     * @param sendName
     * @return
     */
    private Map<String, Object> getParamMap(String orderNNo, String openId, Integer money, String actName, String wishing, String remark, String sendName) {
        Map<String, Object> paramMap = new HashMap<String, Object>();
        paramMap.put("nonce_str", getNonceStr());//随机字符串
        paramMap.put("mch_billno", orderNNo);//商户订单
        paramMap.put("mch_id", CERT_VALUE);//商户号
        paramMap.put("wxappid", APP_ID);//商户appid
        paramMap.put("send_name", sendName);//发红包者名称
        paramMap.put("re_openid", openId);//用户openid
        paramMap.put("total_amount", money);//付款金额
        paramMap.put("total_num", 1);//红包发送总人数
        paramMap.put("wishing", wishing);//红包祝福语
        paramMap.put("client_ip", getIpAddress());//接口调用机器IP地址
        paramMap.put("act_name", actName);//活动名称
        paramMap.put("remark", remark);//备注
        paramMap.put("scene_id", "");//场景id（红包大于200必须填写）
        paramMap.put("risk_info", "");//活动信息
        paramMap.put("consume_mch_id", "");//资金授权商号
        paramMap.put("sign", redSignal(paramMap));//签名
        return paramMap;
    }

    /**
     * 根据参数获取红包签名
     *
     * @param params
     * @return
     */
    private static String redSignal(Map<String, Object> params) {
        SortedMap<String, String> packageParams = new TreeMap<String, String>();
        for (Map.Entry<String, Object> m : params.entrySet()) {
            packageParams.put(m.getKey(), m.getValue().toString());
        }
        StringBuffer sb = new StringBuffer();
        Set<?> es = packageParams.entrySet();
        Iterator<?> it = es.iterator();
        while (it.hasNext()) {
            Map.Entry entry = (Map.Entry) it.next();
            String k = (String) entry.getKey();
            String v = (String) entry.getValue();
            if (!StringUtils.isEmpty(v) && !"sign".equals(k) && !"key".equals(k)) {
                sb.append(k + "=" + v + "&");
            }
        }
        sb.append("key=" + KEY);
        String sign = MD5Util.MD5Encode(sb.toString(), "UTF-8").toUpperCase();
        return sign;
    }

    /**
     * 得到NonceStr随机串
     *
     * @return
     */
    public static String getNonceStr() {
        String uuid = UUID.randomUUID().toString();
        uuid = uuid.substring(0, 8) + uuid.substring(9, 13) + uuid.substring(14, 18) + uuid.substring(19, 23) + uuid.substring(24);
        return uuid;
    }

    /**
     * 得到真实Ip
     *
     * @return
     */
    public String getIpAddress() {
        try {
            return InetAddress.getLocalHost().getHostAddress().toString();
        } catch (UnknownHostException e) {
            e.printStackTrace();
        }
        return "";
    }

    /**
     * AES解密验证是否跟真实数据匹配
     *
     * @return
     */
    public String checkAESData(RedPaper param) {
        // 解密
        String check = AESUtils.decrypt(param.getCheckData(), AesConstant.KEY);
        RedPaper entity = JSON.parseObject(check, RedPaper.class);
        String inviteCode= entity.getInviteCode();
        String clueUuid = entity.getClueUuid();
        String redPaper = entity.getRedPaper();
        String shouldMoney = entity.getShouldMoney();
         // 如果都验证成功则返回成功,如果有不一样则返回失败
        if (param.getInviteCode().equals(inviteCode) && param.getClueUuid().equals(clueUuid) && param.getRedPaper().equals(redPaper) && param.getShouldMoney().equals(shouldMoney)) {
            return "1";
        } else {
            return "-1";
        }

    }


    /**
     * 将红包数据推送到
     *
     * @return
     */
    public void pushRedPaper(RedPaper redPaper) {
        logger.info("开始推送内数据到外");

        //设置请求头
        final Map<String, String> headerMap = Maps.newHashMap();

        headerMap.put("App-Key", InOutConstant.APP_KEY);
        headerMap.put("App-Secrety", InOutConstant.APP_SECRETY);

        String extend1 = "GS_REDPAPER";
        String extend2 = JSON.toJSONString(redPaper);
        Map<String, String> params = new HashMap<>();
        params.put("ocUuid", StringUtil.getUUID());
        params.put("extend1", extend1);
        params.put("extend2", extend2);
        logger.info("接口参数" + params.toString());
        try {
            String result = HttpUtil.request(HttpUtil.HttpMethod.POST, InOutConstant.OPEN_PLATFROM_HOST + InOutConstant.OPEN_PLATFROM_OUT_POST, params, headerMap);
            logger.info("response=" + result);
            ResponseData responseData = JSON.parseObject(result, ResponseData.class);
            if (InOutConstant.SUCCESS_CODE == responseData.getMeta().getCode()) {
                logger.info("调用接口成功");

            } else {
                logger.error("调用接口成功，返回的code是:" + responseData.getMeta().getCode());
            }
        } catch (Throwable t) {
            logger.error("调用接口失败，接口参数：" + params.toString(), t);
        }

    }
   
        List<Menu> treeList = menuList.stream().filter(i -> Ids.compare(i.getMenuParentId(), TOP_LEVEL_NUM) == 0).collect(Collectors.toList());
        treeList.forEach(m -> builderHierarchy(m, menuListByAuth));
        
        /**
     * 构建菜单的子菜单列表
     *
     * @param m    当前菜单
     * @param list 菜单列表
     * @return
     */
    private Menu builderHierarchy(Menu m, List<Menu> list) {
        Assert.notNull(m, "m can not be null");
        Assert.notNull(list, "list can not be null");

        list.stream()
                .filter(i -> Ids.eq(i.getMenuParentId(), m.getMenuId()))
                .forEach(i -> {
                    if (m.getMenuList() == null) {
                        m.setMenuList(Lists.newLinkedList());
                    }
                    m.getMenuList().add(builderHierarchy(i, list));
                });
        return m;
    }
}
