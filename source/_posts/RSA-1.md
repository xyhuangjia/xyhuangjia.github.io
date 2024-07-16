---
title: RSA加密（一）
date: 2016-07-13 10:08:00
tags:
---


近期公司支付接口总是被人公司修改数据，原来的MD5加密已经不能满足与当前的安全要求，于是我们采用了一种更为安全的加密方式RSA+AES加密。在开发过程中由于双方之前都没有直接参入这种加密方式开发，所以我们分别采用RSA和AES接口测试。闲话不多说，开始步入流程：
<!-- more -->
#### 第一步：生成私钥公钥证书
1、生成私钥

```shell
openssl genrsa -out rsa_private_key.pem 1024 
```
2、生成公钥
```shell
openssl rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem
```
3、 由于Java服务器和我们加密解密方式不一样(我们使用PCKS#1,他们使用PCKS#8)为了配合他们我们一般需要导出一个PCKS#8格式的密钥证书(注:证书导出不可逆且公钥无法导出PCKS#8证书,IOS和JAVA服务器可以通用PCKS#1公钥证书)
```shell
openssl pkcs8 -topk8 -inform PEM -in private_rsa.pem -outform PEM -nocrypt -out private_key.pem
```
#### 第二步：导入openssl库
pod入openssl库，没有安装cocopods的自行百度安装方法   

#### 第三步：编写加密解密方法

```objective-c
#import <Foundation/Foundation.h>
#include <openssl/rsa.h>
#include <openssl/pem.h>
#include <openssl/err.h>
#include <openssl/md5.h>

/**
 @abstract  padding type
 */
typedef NS_ENUM(NSInteger, RSA_PADDING_TYPE) {
    
    RSA_PADDING_TYPE_NONE       = RSA_NO_PADDING,
    RSA_PADDING_TYPE_PKCS1      = RSA_PKCS1_PADDING,
    RSA_PADDING_TYPE_SSLV23     = RSA_SSLV23_PADDING
};

@interface BBRSACryptor : NSObject
{
    RSA *_rsaPublic;
    RSA *_rsaPrivate;
    
    @public
    RSA *_rsa;
}
- (NSString *)signString:(NSString *)string;
- (BOOL)verifyMD5String:(NSString *)string withSign:(NSString *)signString;
- (NSString *)signMD5String:(NSString *)string;
- (BOOL)verifyString:(NSString *)string withSign:(NSString *)signString;
/**
 Generate rsa key pair by the key size.
 @param keySize RSA key bits . The value could be `512`,`1024`,`2048` and so on.
 Normal is `1024`.
 */
- (BOOL)generateRSAKeyPairWithKeySize:(int)keySize;

/**
 @abstract  import public key, call before 'encryptWithPublicKey'
 @param     publicKey with base64 encoded
 @return    Success or not.
 */
- (BOOL)importRSAPublicKeyBase64:(NSString *)publicKey;

/**
 @abstract  import private key, call before 'decryptWithPrivateKey'
 @param privateKey with base64 encoded
 @return Success or not.
 */
- (BOOL)importRSAPrivateKeyBase64:(NSString *)privateKey;

/**
 @abstract  export public key, 'generateRSAKeyPairWithKeySize' or 'importRSAPublicKeyBase64' should call before this method
 @return    public key base64 encoded
 */
- (NSString *)base64EncodedPublicKey;

/**
 @abstract  export public key, 'generateRSAKeyPairWithKeySize' or 'importRSAPrivateKeyBase64' should call before this method
 @return    private key base64 encoded
 */
- (NSString *)base64EncodedPrivateKey;

/**
 @abstract  encrypt text using RSA public key
 @param     padding type add the plain text
 @return    encrypted data
 */
- (NSData *)encryptWithPublicKeyUsingPadding:(RSA_PADDING_TYPE)padding
                                   plainData:(NSData *)plainData;

/**
 @abstract  encrypt text using RSA private key
 @param     padding type add the plain text
 @return    encrypted data
 */
- (NSData *)encryptWithPrivateKeyUsingPadding:(RSA_PADDING_TYPE)padding
                                    plainData:(NSData *)plainData;

/**
 @abstract  decrypt text using RSA private key
 @param     padding type add the plain text
 @return    encrypted data
 */
- (NSData *)decryptWithPrivateKeyUsingPadding:(RSA_PADDING_TYPE)padding
                                   cipherData:(NSData *)cipherData;

/**
 @abstract  decrypt text using RSA public key
 @param     padding type add the plain text
 @return    encrypted data
 */
- (NSData *)decryptWithPublicKeyUsingPadding:(RSA_PADDING_TYPE)padding
                                  cipherData:(NSData *)cipherData;
@end
```
```objective-c
#import "BBRSACryptor.h"

#define DocumentsDir [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject]
#define OpenSSLRSAKeyDir [DocumentsDir stringByAppendingPathComponent:@".openssl_rsa"]
#define OpenSSLRSAPublicKeyFile [OpenSSLRSAKeyDir stringByAppendingPathComponent:@"bb.publicKey.pem"]
#define OpenSSLRSAPrivateKeyFile [OpenSSLRSAKeyDir stringByAppendingPathComponent:@"bb.privateKey.pem"]

@implementation BBRSACryptor

- (instancetype)init
{
    self = [super init];
    if (self) {
        
        // mkdir for key dir
        NSFileManager *fm = [NSFileManager defaultManager];
        if (![fm fileExistsAtPath:OpenSSLRSAKeyDir])
        {
            [fm createDirectoryAtPath:OpenSSLRSAKeyDir withIntermediateDirectories:YES attributes:nil error:nil];
        }
    }
    return self;
}
/**
 *  <#Description#>
 *
 *  @param keySize <#keySize description#>
 *
 *  @return <#return value description#>
 */
- (BOOL)generateRSAKeyPairWithKeySize:(int)keySize
{
    if (NULL != _rsa)
    {
        RSA_free(_rsa);
        _rsa = NULL;
    }
    _rsa = RSA_generate_key(keySize,RSA_F4,NULL,NULL);
    assert(_rsa != NULL);
    
    const char *publicKeyFileName = [OpenSSLRSAPublicKeyFile cStringUsingEncoding:NSASCIIStringEncoding];
    const char *privateKeyFileName = [OpenSSLRSAPrivateKeyFile cStringUsingEncoding:NSASCIIStringEncoding];
    
    //写入私钥和公钥
    RSA_blinding_on(_rsa, NULL);
    
    BIO *priBio = BIO_new_file(privateKeyFileName, "w");
    PEM_write_bio_RSAPrivateKey(priBio, _rsa, NULL, NULL, 0, NULL, NULL);
    
    BIO *pubBio = BIO_new_file(publicKeyFileName, "w");
    
    
    PEM_write_bio_RSA_PUBKEY(pubBio, _rsa);
//    PEM_write_bio_RSAPublicKey(pubBio, _rsa);
    
    BIO_free(priBio);
    BIO_free(pubBio);
    
    //分别获取公钥和私钥
    _rsaPrivate = RSAPrivateKey_dup(_rsa);
    assert(_rsaPrivate != NULL);
    
    _rsaPublic = RSAPublicKey_dup(_rsa);
    assert(_rsaPublic != NULL);
    
    NSLog(@"公钥路径:\n %@",OpenSSLRSAPublicKeyFile);
    NSLog(@"私钥路径:\n %@",OpenSSLRSAPrivateKeyFile);
    
    if (_rsa && _rsaPublic && _rsaPrivate)
    {
        return YES;
    }
    else
    {
        return NO;
    }
}
/**
 *  <#Description#>
 *
 *  @param publicKey <#publicKey description#>
 *
 *  @return <#return value description#>
 */
- (BOOL)importRSAPublicKeyBase64:(NSString *)publicKey
{
    //格式化公钥
    NSMutableString *result = [NSMutableString string];
    [result appendString:@"-----BEGIN PUBLIC KEY-----\n"];
    int count = 0;
    for (int i = 0; i < [publicKey length]; ++i) {
        
        unichar c = [publicKey characterAtIndex:i];
        if (c == '\n' || c == '\r') {
            continue;
        }
        [result appendFormat:@"%c", c];
        if (++count == 64) {
            [result appendString:@"\n"];
            count = 0;
        }
    }
    [result appendString:@"\n-----END PUBLIC KEY-----"];
    [result writeToFile:OpenSSLRSAPublicKeyFile
             atomically:YES
               encoding:NSASCIIStringEncoding
                  error:NULL];
    
    FILE *publicKeyFile;
//    NSLog(@"%@",result);
    const char *publicKeyFileName = [OpenSSLRSAPublicKeyFile cStringUsingEncoding:NSASCIIStringEncoding];
    publicKeyFile = fopen(publicKeyFileName,"rb");
    if (NULL != publicKeyFile)
    {
        BIO *bpubkey = NULL;
        bpubkey = BIO_new(BIO_s_file());
        BIO_read_filename(bpubkey, publicKeyFileName);
        
        _rsaPublic = PEM_read_bio_RSA_PUBKEY(bpubkey, NULL, NULL, NULL);
        assert(_rsaPublic != NULL);
        BIO_free_all(bpubkey);
    }
    
    return YES;
}
/**
 *  <#Description#>
 *
 *  @param privateKey <#privateKey description#>
 *
 *  @return <#return value description#>
 */
- (BOOL)importRSAPrivateKeyBase64:(NSString *)privateKey
{
    //格式化私钥
    const char *pstr = [privateKey UTF8String];
    int len = (int)[privateKey length];
   // NSLog(@"%d",len);
    NSMutableString *result = [NSMutableString string];
    [result appendString:@"-----BEGIN RSA PRIVATE KEY-----\n"];
    int index = 0;
    int count = 0;
    while (index < len) {
        char ch = pstr[index];
        if (ch == '\r' || ch == '\n') {
            ++index;
            continue;
        }
        [result appendFormat:@"%c", ch];
        if (++count == 64)
        {
            [result appendString:@"\n"];
            count = 0;
        }
        index++;
    }
    [result appendString:@"\n-----END RSA PRIVATE KEY-----"];
    
    [result writeToFile:OpenSSLRSAPrivateKeyFile
             atomically:YES
               encoding:NSASCIIStringEncoding
                  error:NULL];
//    NSLog(@"%@",result);
    FILE *privateKeyFile;
    const char *privateKeyFileName = [OpenSSLRSAPrivateKeyFile cStringUsingEncoding:NSASCIIStringEncoding];
    privateKeyFile = fopen(privateKeyFileName,"rb");
    if (NULL != privateKeyFile)
    {
        BIO *bpubkey = NULL;
        bpubkey = BIO_new(BIO_s_file());
        
        BIO_read_filename(bpubkey, privateKeyFileName);
       // _rsaPrivate = PEM_read_bio_PrivateKey(bpubkey, NULL, NULL, NULL);
        _rsaPrivate = PEM_read_bio_RSAPrivateKey(bpubkey, NULL, NULL, NULL);
        assert(_rsaPrivate != NULL);
        BIO_free_all(bpubkey);
    }
    
    return YES;
}
/**
 *  <#Description#>
 *
 *  @return <#return value description#>
 */
- (NSString *)base64EncodedPublicKey
{
    NSFileManager *fm = [NSFileManager defaultManager];
    if ([fm fileExistsAtPath:OpenSSLRSAPublicKeyFile])
    {
        //NSLog(@"%@",OpenSSLRSAPublicKeyFile);
        NSString *str = [NSString stringWithContentsOfFile:OpenSSLRSAPublicKeyFile encoding:NSUTF8StringEncoding error:nil];
        NSString *string = [[str componentsSeparatedByString:@"-----"] objectAtIndex:2];
        string = [string stringByReplacingOccurrencesOfString:@"\n" withString:@""];
        string = [string stringByReplacingOccurrencesOfString:@"\r" withString:@""];
        //NSLog(@"%@",string);
        return string;
    }
    return nil;
}
/**
 *  <#Description#>
 *
 *  @return <#return value description#>
 */
- (NSString *)base64EncodedPrivateKey
{
    NSFileManager *fm = [NSFileManager defaultManager];
    if ([fm fileExistsAtPath:OpenSSLRSAPrivateKeyFile])
    {
        NSString *str = [NSString stringWithContentsOfFile:OpenSSLRSAPrivateKeyFile encoding:NSUTF8StringEncoding error:nil];
        NSString *string = [[str componentsSeparatedByString:@"-----"] objectAtIndex:2];
        string = [string stringByReplacingOccurrencesOfString:@"\n" withString:@""];
        string = [string stringByReplacingOccurrencesOfString:@"\r" withString:@""];
        return string;
    }
    return nil;
}
/**
 *  <#Description#>
 *
 *  @param padding   <#padding description#>
 *  @param plainData <#plainData description#>
 *
 *  @return <#return value description#>
 */
- (NSData *)encryptWithPublicKeyUsingPadding:(RSA_PADDING_TYPE)padding plainData:(NSData *)plainData
{
    NSAssert(_rsaPublic != NULL, @"You should import public key first");
    
    if ([plainData length])
    {
        int len = (int)[plainData length];
        unsigned char *plainBuffer = (unsigned char *)[plainData bytes];
        
        //result len
        int clen = RSA_size(_rsaPublic);
        unsigned char *cipherBuffer = calloc(clen, sizeof(unsigned char));
        
        RSA_public_encrypt(len,plainBuffer,cipherBuffer, _rsaPublic,  padding);
        
        NSData *cipherData = [[NSData alloc] initWithBytes:cipherBuffer length:clen];
        
        free(cipherBuffer);
        
        return cipherData;
    }
    
    return nil;
}
/**
 *  <#Description#>
 *
 *  @param padding   <#padding description#>
 *  @param plainData <#plainData description#>
 *
 *  @return <#return value description#>
 */
- (NSData *)encryptWithPrivateKeyUsingPadding:(RSA_PADDING_TYPE)padding plainData:(NSData *)plainData
{
    NSAssert(_rsaPrivate != NULL, @"You should import private key first");
    
    if ([plainData length])
    {
        int len = (int)[plainData length];
        unsigned char *plainBuffer = (unsigned char *)[plainData bytes];
        
        //result len
        int clen = RSA_size(_rsaPrivate);
        unsigned char *cipherBuffer = calloc(clen, sizeof(unsigned char));
        
        RSA_private_encrypt(len,plainBuffer,cipherBuffer, _rsaPrivate,  padding);
        
        NSData *cipherData = [[NSData alloc] initWithBytes:cipherBuffer length:clen];
        
        free(cipherBuffer);
        
        return cipherData;
    }
    
    return nil;
}
/**
 *  <#Description#>
 *
 *  @param padding    <#padding description#>
 *  @param cipherData <#cipherData description#>
 *
 *  @return <#return value description#>
 */
- (NSData *)decryptWithPrivateKeyUsingPadding:(RSA_PADDING_TYPE)padding cipherData:(NSData *)cipherData
{
    NSAssert(_rsaPrivate != NULL, @"You should import private key first");
    
    if ([cipherData length])
    {
        int len = (int)[cipherData length];
        unsigned char *cipherBuffer = (unsigned char *)[cipherData bytes];
        
        //result len
        int mlen = RSA_size(_rsaPrivate);
        unsigned char *plainBuffer = calloc(mlen, sizeof(unsigned char));
        
        RSA_private_decrypt(len, cipherBuffer, plainBuffer, _rsaPrivate, padding);
        
        NSData *plainData = [[NSData alloc] initWithBytes:plainBuffer length:mlen];
        
        free(plainBuffer);
        
        return plainData;
    }
    
    return nil;
}
/**
 *  <#Description#>
 *
 *  @param padding    <#padding description#>
 *  @param cipherData <#cipherData description#>
 *
 *  @return <#return value description#>
 */
- (NSData *)decryptWithPublicKeyUsingPadding:(RSA_PADDING_TYPE)padding cipherData:(NSData *)cipherData
{
    NSAssert(_rsaPublic != NULL, @"You should import public key first");
    
    if ([cipherData length])
    {
        int len = (int)[cipherData length];
        unsigned char *cipherBuffer = (unsigned char *)[cipherData bytes];
        
        //result len
        int mlen = RSA_size(_rsaPublic);
        unsigned char *plainBuffer = calloc(mlen, sizeof(unsigned char));
        
        RSA_public_decrypt(len, cipherBuffer, plainBuffer, _rsaPublic, padding);
        
        NSData *plainData = [[NSData alloc] initWithBytes:plainBuffer length:mlen];
        
        free(plainBuffer);
        
        return plainData;
    }
    
    return nil;
}
#pragma mark RSA sha1验证签名
//signString为base64字符串
- (BOOL)verifyString:(NSString *)string withSign:(NSString *)signString
{
    if (!_rsaPublic) {
        NSLog(@"please import public key first");
        return NO;
    }
    
    const char *message = [string cStringUsingEncoding:NSUTF8StringEncoding];
    int messageLength = (int)[string lengthOfBytesUsingEncoding:NSUTF8StringEncoding];
    NSData *signatureData = [[NSData alloc]initWithBase64EncodedString:signString options:0];
    unsigned char *sig = (unsigned char *)[signatureData bytes];
    unsigned int sig_len = (int)[signatureData length];
    
    
    
    
    unsigned char sha1[20];
    SHA1((unsigned char *)message, messageLength, sha1);
    int verify_ok = RSA_verify(NID_sha1
                               , sha1, 20
                               , sig, sig_len
                               , _rsaPublic);
    
    if (1 == verify_ok){
        return   YES;
    }
    return NO;
    
    
}
#pragma mark RSA MD5 验证签名
- (BOOL)verifyMD5String:(NSString *)string withSign:(NSString *)signString
{
    if (!_rsaPublic) {
        NSLog(@"please import public key first");
        return NO;
    }
    
    const char *message = [string cStringUsingEncoding:NSUTF8StringEncoding];
    // int messageLength = (int)[string lengthOfBytesUsingEncoding:NSUTF8StringEncoding];
    NSData *signatureData = [[NSData alloc]initWithBase64EncodedString:signString options:0];
    unsigned char *sig = (unsigned char *)[signatureData bytes];
    unsigned int sig_len = (int)[signatureData length];
    
    unsigned char digest[MD5_DIGEST_LENGTH];
    MD5_CTX ctx;
    MD5_Init(&ctx);
    MD5_Update(&ctx, message, strlen(message));
    MD5_Final(digest, &ctx);
    int verify_ok = RSA_verify(NID_md5
                               , digest, MD5_DIGEST_LENGTH
                               , sig, sig_len
                               , _rsaPublic);
    if (1 == verify_ok){
        return   YES;
    }
    return NO;
    
}

- (NSString *)signString:(NSString *)string
{
    if (!_rsaPrivate) {
        NSLog(@"please import private key first");
        return nil;
    }
    const char *message = [string cStringUsingEncoding:NSUTF8StringEncoding];
    int messageLength = (int)strlen(message);
    unsigned char *sig = (unsigned char *)malloc(256);
    unsigned int sig_len;
    
    unsigned char sha1[20];
    SHA1((unsigned char *)message, messageLength, sha1);
    
    int rsa_sign_valid = RSA_sign(NID_sha1
                                  , sha1, 20
                                  , sig, &sig_len
                                  , _rsaPrivate);
    if (rsa_sign_valid == 1) {
        NSData* data = [NSData dataWithBytes:sig length:sig_len];
        
        NSString * base64String = [data base64EncodedStringWithOptions:0];
        free(sig);
        return base64String;
    }
    
    free(sig);
    return nil;
}
/**
 *  <#Description#>
 *
 *  @param string <#string description#>
 *
 *  @return <#return value description#>
 */
- (NSString *)signMD5String:(NSString *)string
{
    if (!_rsaPrivate) {
        NSLog(@"please import private key first");
        return nil;
    }
    const char *message = [string cStringUsingEncoding:NSUTF8StringEncoding];
    //int messageLength = (int)strlen(message);
    unsigned char *sig = (unsigned char *)malloc(256);
    unsigned int sig_len;
    
    unsigned char digest[MD5_DIGEST_LENGTH];
    MD5_CTX ctx;
    MD5_Init(&ctx);
    MD5_Update(&ctx, message, strlen(message));
    MD5_Final(digest, &ctx);
    
    int rsa_sign_valid = RSA_sign(NID_md5
                                  , digest, MD5_DIGEST_LENGTH
                                  , sig, &sig_len
                                  , _rsaPrivate);
    
    if (rsa_sign_valid == 1) {
        NSData* data = [NSData dataWithBytes:sig length:sig_len];
        
        NSString * base64String = [data base64EncodedStringWithOptions:0];
        free(sig);
        return base64String;
    }
    
    free(sig);
    return nil;
    
    
}

@end


```

加密签名文件和解密验签文件
```objective-c
#import "BBRSACryptor.h"
#import "GTMBase64.h"

@interface BBRSACryptor (XHCategory)

/**
 *  生成公钥,私钥 (生成成功后控制台会打印出 公钥,私钥 存储路径)
 */
+(void)createPublicKeyAndPrivateKey;

/**
 *  公钥加密
 *
 *  @param string    普通字符串
 *  @param publicKey 公钥
 *
 *  @return 加密后字符串
 */
+(NSString *)encryptString:(NSString *)string publicKey:(NSString *)publicKey;

/**
 *  公钥解密
 *
 *  @param string    私钥加密字符串
 *  @param publicKey 公钥
 *
 *  @return 解密后字符串
 */
+(NSString *)decodingString:(NSString *)string publicKey:(NSString *)publicKey;

/**
 *  私钥加密
 *
 *  @param string     普通字符串
 *  @param privateKey 私钥
 *
 *  @return 加密后字符串
 */
+(NSString *)encryptString:(NSString *)string privateKey:(NSString *)privateKey;

/**
 *  私钥解密
 *
 *  @param string     公钥加密字符串
 *  @param privateKey 私钥
 *
 *  @return 解密后字符串
 */
+(NSString *)decodingString:(NSString *)string privateKey:(NSString *)privateKey;

/**
 *  私钥签名
 *
 *  @param string     普通字符串
 *  @param privateKey 私钥
 *
 *  @return 签名后字符串
 */
+(NSString *)singString:(NSString *)string privateKey:(NSString *)privateKey;

/**
 *  私钥签名MD5
 *
 *  @param string     普通字符串
 *  @param privateKey 私钥
 *
 *  @return 签名后字符串
 */
+(NSString *)singMD5String:(NSString *)string privateKey:(NSString *)privateKey;

/**
 *  RSA sha1 验证签名
 *
 *  @param string     普通字符串
 *  @param signString 签名字符串(base64)
 *  @param publicKey  公钥
 *
 *  @return 验证结果
 */
+(BOOL)verifyString:(NSString *)string sign:(NSString *)signString publicKey:(NSString *)publicKey;

/**
 *  RSA MD5 验证签名
 *
 *  @param string     普通字符串
 *  @param signString 签名字符串
 *  @param publicKey  公钥
 *
 *  @return 验证结果
 */
+(BOOL)verifyMD5String:(NSString *)string sign:(NSString *)signString publicKey:(NSString *)publicKey;
@end
```
```
#import "BBRSACryptor+XHAdd.h"

@implementation BBRSACryptor (XHCategory)

/**
 *  生成公钥,私钥
 */
+(void)createPublicKeyAndPrivateKey
{
    BBRSACryptor *reaCryptor = [[BBRSACryptor alloc] init];
    [reaCryptor generateRSAKeyPairWithKeySize:1024];
}
/**
 *  公钥加密
 *
 *  @param string    普通字符串
 *  @param publicKey 公钥
 *
 *  @return 加密后字符串
 */
+(NSString *)encryptString:(NSString *)string publicKey:(NSString *)publicKey
{
    BBRSACryptor *rsaCryptor = [[BBRSACryptor alloc] init];
    if([rsaCryptor importRSAPublicKeyBase64:publicKey])
    {
        NSData *cipherData = [rsaCryptor encryptWithPublicKeyUsingPadding:RSA_PADDING_TYPE_PKCS1 plainData:[string dataUsingEncoding:NSUTF8StringEncoding]];
        NSString *cipherString = [GTMBase64 stringByEncodingData:cipherData];
        return cipherString;
    }
    return nil;
}

/**
 *  公钥解密
 *
 *  @param string    私钥加密字符串
 *  @param publicKey 公钥
 *
 *  @return 解密后字符串
 */
+(NSString *)decodingString:(NSString *)string publicKey:(NSString *)publicKey
{
    BBRSACryptor *rsaCryptor = [[BBRSACryptor alloc] init];
    if([rsaCryptor importRSAPublicKeyBase64:publicKey])
    {
        NSData *cipherData = [GTMBase64 decodeString:string];
        NSData *plainData =  [rsaCryptor decryptWithPublicKeyUsingPadding:RSA_PADDING_TYPE_PKCS1 cipherData:cipherData];
        NSString *plainStr = [[NSString alloc]initWithData:plainData encoding:NSUTF8StringEncoding];
        return plainStr;
    }
    return nil;
}

/**
 *  私钥加密
 *
 *  @param string     普通字符串
 *  @param privateKey 私钥
 *
 *  @return 加密后字符串
 */
+(NSString *)encryptString:(NSString *)string privateKey:(NSString *)privateKey
{
    BBRSACryptor *rsaCryptor = [[BBRSACryptor alloc] init];
    if([rsaCryptor importRSAPrivateKeyBase64:privateKey])
    {
        NSData *cipherData = [rsaCryptor encryptWithPrivateKeyUsingPadding:RSA_PKCS1_PADDING plainData:[string dataUsingEncoding:NSUTF8StringEncoding]];
        NSString *cipherString = [GTMBase64 stringByEncodingData:cipherData];
        return cipherString;
    }
    return nil;
}

/**
 *  私钥解密
 *
 *  @param string     公钥加密字符串
 *  @param privateKey 私钥
 *
 *  @return 解密后字符串
 */
+(NSString *)decodingString:(NSString *)string privateKey:(NSString *)privateKey
{
    BBRSACryptor *rsaCryptor = [[BBRSACryptor alloc] init];
    if([rsaCryptor importRSAPrivateKeyBase64:privateKey])
    {
        NSData *cipherData = [GTMBase64 decodeString:string];
        NSData *plainData = [rsaCryptor decryptWithPrivateKeyUsingPadding:RSA_PADDING_TYPE_PKCS1 cipherData:cipherData];
        NSString *plainText = [[NSString alloc]initWithData:plainData encoding:NSUTF8StringEncoding];
        return plainText;
    }
    return nil;
}

/**
 *  私钥签名
 *
 *  @param string     普通字符串
 *  @param privateKey 私钥
 *
 *  @return 签名后字符串
 */
+(NSString *)singString:(NSString *)string privateKey:(NSString *)privateKey
{

    BBRSACryptor *rsaCryptor = [[BBRSACryptor alloc] init];
    if([rsaCryptor importRSAPrivateKeyBase64:privateKey])
    {
       NSString* sing= [rsaCryptor signString:string];
        return  sing;
    }
    return nil;
}

/**
 *  私钥签名MD5
 *
 *  @param string     普通字符串
 *  @param privateKey 私钥
 *
 *  @return 签名后字符串
 */
+(NSString *)singMD5String:(NSString *)string privateKey:(NSString *)privateKey
{

    BBRSACryptor *rsaCryptor = [[BBRSACryptor alloc] init];
    if([rsaCryptor importRSAPrivateKeyBase64:privateKey])
    {
         NSString* singMd5 = [rsaCryptor signMD5String:string];
         return  singMd5;
    }
    return nil;
}

/**
 *  RSA sha1 验证签名
 *
 *  @param string     普通字符串
 *  @param signString 签名字符串(base64)
 *  @param publicKey  公钥
 *
 *  @return 验证结果
 */
+(BOOL)verifyString:(NSString *)string sign:(NSString *)signString publicKey:(NSString *)publicKey
{
    BBRSACryptor *rsaCryptor = [[BBRSACryptor alloc] init];
    if([rsaCryptor importRSAPublicKeyBase64:publicKey])
    {
     return [rsaCryptor verifyString:string withSign:signString];
    }
    return NO;
}

/**
 *  RSA MD5 验证签名
 *
 *  @param string     普通字符串
 *  @param signString 签名字符串
 *  @param publicKey  公钥
 *
 *  @return 验证结果
 */
+(BOOL)verifyMD5String:(NSString *)string sign:(NSString *)signString publicKey:(NSString *)publicKey
{
    BBRSACryptor *rsaCryptor = [[BBRSACryptor alloc] init];
    if([rsaCryptor importRSAPublicKeyBase64:publicKey])
    {
        return [rsaCryptor verifyMD5String:string withSign:signString];
    }
    return NO;
}
@end
```
#### 第四步：使用案例

```
  NSString* private_key_string = @"MIICdwIBADANBgkqhkiG9w0BAQEFAASCAmEwggJdAgEAAoGBANv2AQ9nX46HX/+tIyi/j7L62z+jb4dy009+vADUg6F5oBGkDcdPLudcSQzP04iZLJifqWtngUoZ8Pc5WDDuaV4UvOsGzf8IjqzTGZD8eljKKxA+aV5HaMOp/7zTvSBzq0nGcuOWfAERk8L78/EpHwfAyM2XE9u3M2En4+HIkeq3AgMBAAECgYEAm/CV49PHnQZAesTGTlcwixTpZv55TS+Mu6j/pB8Fiu7tGlSSKCDtAb0dVOXp88eUJEfdFnX05RHrEXooGdiL/YQ1KaKcg2N25i9nn+vTQdgBZ64+HR0G47yrAMzvvOwCA7zZVK1+WF+DoK9M9tU7PRPOxMWNgHXPNq6IW21CGgECQQD3mxCofN0jbwWe49P7wEzz6tS2cmGwF0cAGy7zZdsPATs7Bvd/xQvb3/dUVFep/tado4bCi2bg+jQuzf5j4tM3AkEA42sDmDbDGmniWGZcdjaBVWl7Mbba8QTlO4lvS/1tgayU7QG0k+Hp4dpdE9E+LL3cZo3n9Zc+rbHIwr3JF1xkgQJAXVh9QDfKmqgpQ0x6x2co27AFPz8B6wPrhXO6EJKusgpxzQAEYIvlu5/Eu2sMnY7wU/+pN0CcqWZKM/b+16NUowJAZspZ55To/qlZS0eJB01/i9GPg1r4/vONgSmPirNTqccN0UpyCl2UTydZ5rku9x4h3qDJdXIVPIEdExihKdPzAQJBANx8nh0veU4ok1OFZnm8SUfZXMYc05g3iw2Q+1ZajCkFco4o1pXIIGn72QnuWJ18CvfnXTDquw826eaHWtlgis0=";
    
    
    NSString* public_key_string = @"MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDh5nxZMZ/lCttyHyrOh5AImOUh5OyATJ8fB5z4WlvBCxpe0rUAQ1VQfzOArxB+B4YUxokNijJxwpSiEYvfRk2Xz0I2/LxMq1g+8Stv6SPj4pe2NZRut5NLxLaihtb4Gfuw4GanX5bLauC7BY1akxyCSu0mRpFZ0nNHSuPnCzUHlQIDAQAB";

    NSDictionary *stuDic = [NSDictionary dictionaryWithObjectsAndKeys:
                            @"小华",@"userAccount",
                            @"123456",@"phoneCode",
                            @"123456",@"password",
                            @"1",@"AppTye",
                            nil];
    NSString *tempStr = [@"1|" stringByAppendingString:[self dictionaryToJson:stuDic]];
    //NSString * jsonString =[self HAReplaceString:[NSString stringWithFormat:@"1|%@",[self dictionaryToJson:stuDic]] excuseString:@" " replaceSting:@""] ;
    //格式化
    NSString *str = [self HAReplaceString:tempStr excuseString:@" " replaceSting:@""];
    
//     NSString * jsonString = @"123456";
    NSString * jsonString = [self HAReplaceString:str excuseString:@"\n" replaceSting:@""];
    NSLog(@"jsonString==%@",jsonString);
//    NSData * testData = [GTMBase64 decodeString:public_key_string];
//    NSString * testString = [[NSString alloc]initWithData:testData encoding:NSUTF8StringEncoding];
//    NSLog(@"testString===%@",testString);
    NSString * enString =[BBRSACryptor encryptString:jsonString publicKey:public_key_string];
//    NSLog(@"加密==\n%@",enString);
//    NSString * deString =[BBRSACryptor decodingString:enString privateKey:private_key_string];
//     NSLog(@"解密===\n%@",deString);
    NSString * sign = [BBRSACryptor singString:jsonString privateKey:private_key_string];
//    NSLog(@"签名:\n%@",sign);
//    BOOL match = [BBRSACryptor verifyString:jsonString sign:sign publicKey:public_key_string];
//    NSLog(@"验签==%d",match);
//    NSDictionary * dic =@{@"param":[NSString stringWithFormat:@"%@|%@",enString,sign]};
    NSDictionary * dic =@{@"param":[NSString stringWithFormat:@"%@",sign],@"paramjson":jsonString};
       NSLog(@"dic==%@===",dic);
    [HABaseRequest requestWithURLName:@"register/registV"
                            Parameter:dic
                         SuccessBlock:^(id returnValue) {
                             
                         } FailBlock:^(NSError *error) {
                             
                         }];
```
[参考github源码]( https://github.com/CoderZhuXH/BBRSACryptor-XHAdd)