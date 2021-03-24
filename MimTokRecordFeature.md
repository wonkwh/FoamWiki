# Mimtok Record Feature
- `MHBeauty`, `TXLiteAVSDK_Professional` 가 사용된 소스 분석

- `TCVideoRecordViewController` 가 핵심 viewController 
   - 89, 90 line에 

```
@property(nonatomic,strong)MHMeiyanMenusView *menusView;
@property(nonatomic,strong)MHBeautyManager *beautyManager;
```
- TXVideoCustomProcessDelegate
   - onPreProcessTexture
      - [self.beautyManager processWithTexture:texture width:width height:height];
- isTxFilter 가 no 
   - common 에 getTISDKKey 
      -  userDefault 에 sprout_key 
   - menus

### default 로 beauty filter 적용시키기 
- beauty 적용시키면 
   - MHMeiyanMenusViewDelegate 
      - - (void)beautyEffectWithLevel:(NSInteger)beauty whitenessLevel:(NSInteger)white ruddinessLevel:(NSInteger)ruddiness
         -    [[TXUGCRecord shareInstance] setBeautyStyle:0 beautyLevel:beauty whitenessLevel:white ruddinessLevel:ruddiness];
         -    러블리 적용시 
            -    beauty : 5, white: 5, ruddiness : 0
            -    인스타 적용시 5,9,7

### 같이찍기 
- 
## reference
- 