---
title: Oracle ERP(EBS) 服务化助力互联网企业应对业务快速变化
date: 2018-12-17 00:00:00
author: robin zheng
categories: 套装软件
tags:
  - ERP
  - SOA
  - ESB
---

#### 在GV，我们采用将大核心ERP系统全盘服务化，和MES，DES等关键业务系统通过服务做流程编排，实现灵动的敏捷。

#### 因为信息保密只能通过网上一篇文章简单介绍（Oracle ISG）大致的情况。

<p>Oracle EBS如何与第三方系统相集成？比如这样的需求，X系统知道物料编码，需要从EBS系统里读取具体物料信息，或者X系统想把自己的人员信息同步到EBS，这类集成问题你就可能需要用到<a href="http://blog.csdn.net/pan_tian/article/details/8504882" rel="nofollow">Oracle EBS Integrated SOA Gateway</a>。</p>
<p>Integrated SOA Gateway是EBS里的一个职责，<a href="http://blog.csdn.net/pan_tian/article/details/8504882" rel="nofollow">分配给用户后就能看到</a> ，进入职责（如下图）后，就能看到所有Oracle EBS可以（只是可以，真正放开需要发布和部署的动作）对外开发的接口。</p>
<p>（当然如果这些系统自带接口还不能满足你的集成需求的话，那么你就需要<a href="http://blog.csdn.net/pan_tian/article/details/11257993" rel="nofollow">自定义客户化接口</a>了）</p>
<p><img src="http://img.my.csdn.net/uploads/201301/15/1358221280_3240.jpg" alt="" height="408" width="734" /></p>
<p>首先分享一些收集到的文档，后续有需要可以再详细阅读：</p>
<p><a href="http://docs.oracle.com/cd/B34956_01/current/acrobat/120irepug.pdf" rel="nofollow">Oracle® Integration Repository User's Guide</a><br /></p>
<p><a href="http://docs.oracle.com/cd/E18727_01/doc.121/e12064/toc.htm" rel="nofollow">Oracle E-Business Suite Integrated SOA Gateway User's Guide</a><br /><a href="http://docs.oracle.com/cd/B53825_08/current/acrobat/121isgig.pdf" rel="nofollow">Oracle E-Business Suite Integrated SOA Gateway Implementation Guide R12.1</a><br /><a href="http://docs.oracle.com/cd/E18727_01/doc.121/e12065/toc.htm" rel="nofollow">Oracle E-Business Suite Integrated SOA Gateway Developer's Guide</a><br /></p>
<p><a href="http://www.oracleappshub.com/ebs-suite/20-minute-guide-to-oracle-integration-repository/" rel="nofollow">20 Minute Guide to Oracle Integration Repository</a></p>
<p><a href="http://blog.csdn.net/pan_tian/article/details/10097815" rel="nofollow">Enable Oracle E-Business Suite Integrated SOA Gateway </a><br /><a href="http://blog.csdn.net/pan_tian/article/details/8504882" rel="nofollow">Oracle Integration Repository </a><br /></p>
<p><a href="http://docs.oracle.com/cd/E11036_01/integrate.1013/b28994.pdf" rel="nofollow">Oracle Application Server Adapters for Files, FTP, Databases, and Enterprise Messaging User's Guide</a> <br /><a href="https://blogs.oracle.com/stevenChan/entry/securing_e-business_suite_web_services_with_integr" rel="nofollow">Securing E-Business Suite Web Services with Integrated SOA Gateway</a></p>
<p><a href="http://blog.vennster.nl/2012/01/using-soa-gateway-in-ebs-12.html" rel="nofollow">Using SOA Gateway in EBS 12 </a><br /><br /></p><h2>Enable Oracle E-Business Suite Integrated SOA Gateway </h2>
<p>要使用Integrated SOA Gateway，首先要打一些补丁，具体方法可以参见Note<span style="font-family:'Times New Roman';"> <a href="https://support.oracle.com/epmos/faces/DocumentDisplay?_afrLoop=763398313342465&amp;id=556540.1&amp;_afrWindowMode=0&amp;_adf.ctrl-state=vdjoq3p70_4#install1212" rel="nofollow">556540.1</a></span><a href="https://support.oracle.com/epmos/faces/DocumentDisplay?_afrLoop=763398313342465&amp;id=556540.1&amp;_afrWindowMode=0&amp;_adf.ctrl-state=vdjoq3p70_4#install1212" rel="nofollow"> </a>或者 我的<a href="http://blog.csdn.net/pan_tian/article/details/10097815" rel="nofollow">另一篇文章</a>。<br /></p>
<p>普通的用户只能查看EBS的接口信息，但并不能发布接口，以及部署，只有sysadmin账户有这个权限。（注：默认情况下，绝大多数的接口是没有发布及部署的，只有需要sysadmin发布后，第三方系统才能调用。）</p>
<p>如果不想使用sysadmin这个账户来管理接口，那么就要给目标用户分配三个角色，才能做sysadmin同样的事情，这三个角色是：</p>
<p>* Irep Administrator（中文：<span id="N1:DisplayName:0" title="名称">Irep 管理员</span>）<br />* System Integration Developer（中文：<span id="AccessRoleTableRN:AccessRoleNameRN:1"><span id="AccessRoleTableRN:AccessRoleDisplayName:1" title="访问职责">系统集成开发员</span></span>）<br />* System Integration Analyst（中文：<span id="AccessRoleTableRN:AccessRoleNameRN:0"><span id="AccessRoleTableRN:AccessRoleDisplayName:0" title="访问职责">系统集成分析专家</span></span>）<br /><br />每一个角色都有着对接口库不同的权限，如下图</p>
<p><img src="https://img-blog.csdn.net/20130822100526500" alt="" /></p>
<p>sysadmin默认是具有上述三个角色的，所以不需要特别设置。其他用户就需要设置了，如何给用户分配角色可以参见Note:<a href="https://support.oracle.com/epmos/faces/DocumentDisplay?_afrLoop=936970254370630&amp;id=861982.1&amp;_afrWindowMode=0&amp;_adf.ctrl-state=17nubdxmt4_17" rel="nofollow">861982.1</a><br /></p>
<p><br /></p><h2>Interfaces<br /></h2>
<p>接下来会以系统默认提供的一个接口为例，演示一下，如何放开接口，允许外部程序调用的一个过程。<br /></p>
<p>以sysadmin账户登录，然后路径：Integrated SOA Gateway &gt; Supply Chain Management &gt; <span id="ProductFamilyTree40" class="x5f"><span class="xd" title="Product">Inventory</span></span> &gt; <span id="ProductFamilyTree49" class="x5g"><span class="xd">Inventory Organization Setup</span></span>，可以看到两个接口，其中的一个叫Locator Maintenance API，这个接口允许用户创建，修改，删除Locator。</p>
<p>(如果接口的Internal Name为User_XXX_XXX的话，比如USER_PKG_LOT，USER_PKG_SERIAL，表示这个接口是需要用户自定义接口，需要自己实现后台的业务处理逻辑)<br /></p>
<p><img src="https://img-blog.csdn.net/20130821195010437?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGFuX3RpYW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="" /><br /></p>
<p>再点进去就能看到接口的详细信息。你能看到这个接口是一个PL/SQL接口，后台对应的Package叫INV_LOC_WMS_PUB，这个接口包含四个Locator处理的方法。如果你有兴趣，你可以打开INV_LOC_WMS_PUB这个Package，就能看到具体的实现逻辑。</p>
<p><img src="https://img-blog.csdn.net/20130821195237687?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGFuX3RpYW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="" /><br /></p>
<p>每一个方法能点进去，就是这个接口方法的描述以及接口参数。</p>
<p><img src="https://img-blog.csdn.net/20130821195750812?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGFuX3RpYW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="" /><br /></p>
<p><br /></p><h2>生成WSDL</h2>
<p>这里需要把PL/SQL的Package以<a href="http://blog.csdn.net/pan_tian/article/details/10008893" rel="nofollow">Webservice</a>方式发布，就需要把这个接口生成<a href="http://blog.csdn.net/pan_tian/article/details/10008893" rel="nofollow">WSDL</a>。（如何你自己的客户化PL/SQL注册到Integrated SOA Gateway，见<a href="http://blog.csdn.net/pan_tian/article/details/11257993" rel="nofollow">这篇文章</a>）<br /></p>
<p>WSDL: Web服务定义语言（Web Service Definition Language），用来定义服务接口。实际上，它能描述服务的两个不同方面：服务的签名（名字和参数），以及服务的绑定和部署细节（协议和位置）。<br /></p>
<p><img src="https://img-blog.csdn.net/20130821200055281?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGFuX3RpYW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="" /><br /></p>
<p>WSDL生成后，Web Service Status就变成了'Generated'。并且你可以查看WSDL<br /></p>
<p><img src="https://img-blog.csdn.net/20130821200741281?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGFuX3RpYW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="" /><br /></p>
<p>‘View WSDL‘就能看到WSDL信息<br /></p>
<p><img src="https://img-blog.csdn.net/20130821201123468?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGFuX3RpYW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="" /><br /></p>
<p><br /></p><h2>Deploy Webservice </h2>
<p>生成了WSDL后，外部系统还不能调用它，你需要把生成的WSDL部署到应用服务器上，这一步还会安装一些必要的Webservice文件到应用服务器上。这里需要对Webservice的安全类型做一个勾选:Username Token、SAML Token。</p>
<p>关于这两个安全类型，可以详细阅读下<a href="https://blogs.oracle.com/stevenChan/entry/securing_e-business_suite_web_services_with_integr" rel="nofollow">Securing E-Business Suite Web Services with Integrated SOA Gateway</a> ，里边有详细的解释。<br /></p>
<p><img src="https://img-blog.csdn.net/20130821201623875?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGFuX3RpYW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="" /><br /></p>
<p><br /></p><h2>Create Grant</h2>
<p>部署完后，Webservice状态变为了Deployed。这之后还有一步- 授权，你是希望所有用户都能访问接口，还是只是特定用户。<br /></p>
<p><img src="https://img-blog.csdn.net/20130821202440562?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGFuX3RpYW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="" /><br /></p><h2>Test Web service</h2>
<p>WSDL的地址是http://[host]:[port]/webservices/SOAProvider/plsql/inv_loc_wms_pub/?wsdl<br /></p>
<p>那么Oracle自带测试web service的地址是上边WSDL的地址，但是去掉'<strong>?wsdl</strong>' (主要不要去掉斜杠/，否则会报No WebService Provider is registered at this URL)<br /></p>
<p><strong>http://[host]:[port]/webservices/SOAProvider/plsql/inv_loc_wms_pub/</strong></p>
<p><img src="https://img-blog.csdn.net/20130822104115375?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGFuX3RpYW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="" /></p>
<p><img src="https://img-blog.csdn.net/20130822104309796?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGFuX3RpYW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="" /></p>
<p>Invoke后，就把HTML表单上的信息组成Soap报文，发送到应用服务器，服务器会把返回信息也以Soap报文的形式返回给客户端。</p>
<p>下为SOAP请求报文样例：</p>
<p></p><pre><code class="language-html">&lt;soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd"&gt;
	&lt;soap:Header xmlns:ns1="http://xmlns.oracle.com/apps/cux/soaprovider/plsql/cux_0_ws_server_prg/"&gt;
		&lt;ns1:SOAHeader&gt;
			&lt;ns1:Responsibility&gt;INVENTORY&lt;/ns1:Responsibility&gt;
			&lt;ns1:RespApplication&gt;CUX&lt;/ns1:RespApplication&gt;
			&lt;ns1:SecurityGroup&gt;STANDARD&lt;/ns1:SecurityGroup&gt;
			&lt;ns1:NLSLanguage&gt;SIMPLIFIED CHINESE&lt;/ns1:NLSLanguage&gt;
			&lt;ns1:Org_Id&gt;0&lt;/ns1:Org_Id&gt;
		&lt;/ns1:SOAHeader&gt;
		&lt;wsse:Security xmlns:wsse="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd" xmlns="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd" xmlns:env="http://schemas.xmlsoap.org/soap/envelope/" soap:mustUnderstand="1"&gt;
			&lt;wsse:UsernameToken xmlns:wsse="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd" xmlns="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd"&gt;
				&lt;wsse:Username&gt;ESB_TEST&lt;/wsse:Username&gt;
				&lt;wsse:Password Type="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-username-token-profile-1.0#PasswordText"&gt;1234567890&lt;/wsse:Password&gt;
			&lt;/wsse:UsernameToken&gt;
		&lt;/wsse:Security&gt;
	&lt;/soap:Header&gt;
	&lt;soap:Body xmlns:ns2="http://xmlns.oracle.com/apps/cux/soaprovider/plsql/cux_0_ws_server_prg/invokews/"&gt;
		&lt;ns2:InputParameters&gt;
			&lt;ns2:P_IFACE_CODE&gt;XXX&lt;/ns2:P_IFACE_CODE&gt;
			&lt;ns2:P_BATCH_NUMBER&gt;1234567654323&lt;/ns2:P_BATCH_NUMBER&gt;
			&lt;ns2:P_REQUEST_DATA&gt;32424&lt;/ns2:P_REQUEST_DATA&gt;
		&lt;/ns2:InputParameters&gt;
	&lt;/soap:Body&gt;
&lt;/soap:Envelope&gt;
</code></pre>
<p>（请求报文头中的Responsibility,RespApplication,SecurityGroup,NLSLanguage,Org_Id节点都是Oracle EBS系统要求的SOAP头信息，用于账户验证）     <br /><br /></p> 
<p>SOAP响应报文样例：</p> 
<p> </p>
<p></p><pre><code class="language-html">&lt;env:Envelope xmlns:env="http://schemas.xmlsoap.org/soap/envelope/"&gt;
	&lt;env:Header/&gt;
	&lt;env:Body&gt;
		&lt;OutputParameters xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://xmlns.oracle.com/apps/cux/soaprovider/plsql/cux_0_ws_server_prg/invokews/"&gt;
			&lt;X_RETURN_CODE xmlns="http://xmlns.oracle.com/apps/cux/soaprovider/plsql/cux_0_ws_server_prg/invokefmsws/"&gt;ERROR001&lt;/X_RETURN_CODE&gt;
			&lt;X_RETURN_MESG xmlns="http://xmlns.oracle.com/apps/cux/soaprovider/plsql/cux_0_ws_server_prg/invokefmsws/"&gt;输入的请求报文格式不正确&lt;/X_RETURN_MESG&gt;
			&lt;X_RESPONSE_DATA xsi:nil="true" xmlns="http://xmlns.oracle.com/apps/cux/soaprovider/plsql/cux_0_ws_server_prg/invokews/"/&gt;
		&lt;/OutputParameters&gt;
	&lt;/env:Body&gt;
&lt;/env:Envelope&gt;
</code></pre><br />
<p></p>
<p>除了这个标准的测试方法外，也可以使用<a href="http://www.soapui.org/" rel="nofollow">SoapUI</a>这样的专业工具来测试Webservice，见<a href="http://blog.csdn.net/pan_tian/article/details/10301197" rel="nofollow">这篇文章</a>。<br /></p>
<p><br /></p><h2>SOA Monitor</h2>
<p>可以在SOA Monitor中监视客户端调用EBS的Webservice的执行的效果。</p>
<p>路径：Integrated SOA Gateway –&gt; SOA Monitor</p>
<p><img src="https://img-blog.csdn.net/20130825135819375?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGFuX3RpYW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="" /><br /></p>
<p>如果调用失败，可以从详细信息或者日志里查看具体内容。</p>
<p><br /></p>
<p>转载请注明出处：<a href="http://blog.csdn.net/pan_tian/article/details/10159935" rel="nofollow">http://blog.csdn.net/pan_tian/article/details/10159935</a><br /></p>
<p><br /></p>

