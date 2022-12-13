# todayreciption
----index.tsx

----------------

import React from "react";
import Head from 'next/head';
import * as Layout from "../../../components/Layout/index";
import { useDispatch } from "react-redux";
import Grid from '@mui/material/Grid';
import Button from "@mui/material/Button";
import Box from '@mui/material/Box';
import * as store from '../../../stores/Store';

import * as commonTips from "../../../constants/CommonTips";
import { StyledHeader, StyledTitleHeader } from "../../../styles/Styled";

import TipsIcon from "../../../components/Common/Tips";
import DataGrid_list from "../../../components/TodayReception/Datagrid";
import { InputSelect } from "../../../components/Atoms/Select/Select";
import { useCompleteClassList } from "../../../features/CompleteClass/Selector";
import { useTodayReceptionConditions, useTodayReceptionList } from "../../../features/TodayReception/Selector";
import { fetchAsyncTodayReceptionList } from "../../../features/TodayReception/Operation";
import { fetchAsyncCompleteClassList } from "../../../features/CompleteClass/Operation";
import todayReceptionSlice from "../../../features/TodayReception/Slice";
import { TodayReceptionDetail } from "../../../features/TodayReception/Type";

export const Main = ():JSX.Element => {
  // useDispatch で store に紐付いた dispatch が取得できます
  const dispatch: store.AppDispatch = useDispatch();
  const conditions = useTodayReceptionConditions();
  const completeClassList = useCompleteClassList();
  const todayReceptionList = useTodayReceptionList();

  // useEffectとは、関数の実行タイミングをReactのレンダリング後まで遅らせるhook
  React.useEffect(() => {
    /// 第1引数には実行させたい副作用関数を記述
    /// 第2引数には副作用関数の実行タイミングを制御する依存データを記述
    const promise = async() => {
      await dispatch(fetchAsyncCompleteClassList(conditions.title));
      await dispatch(fetchAsyncTodayReceptionList(conditions));
    };      
    promise();
  },[dispatch])

  // ボタンテキスト変更制御
  const receptionButtonText = conditions.isShowReception? "担当顧客分のみ表示" : "受付分全て表示";

  // 完了区分コンボボックス
  const handleOnChangeCompleteKbn = (key:string,column:string,value:string | number) => {
    const newConditions = {...conditions,completeClass:value.toString()};

    // 選択された完了区分でフィルター
    dispatch(todayReceptionSlice.actions.conditionsChanged(newConditions));
  };

  // 受付分全て表示（担当顧客分のみ表示）ボタン押下時
  const handleIsShowReception = (value:boolean) => {
    const newConditions = {...conditions,isShowReception: !(conditions.isShowReception)};
    dispatch(todayReceptionSlice.actions.conditionsChanged({...conditions,isShowReception: !(conditions.isShowReception)})); 

    // リクエスト処理
    const promise = async() => {
      await dispatch(fetchAsyncTodayReceptionList(newConditions));
    };
    promise();
  };

  return(
    <React.Fragment>
      <Head>
        <title>本日応対</title>
      </Head>
      <br></br>
      <br></br>
      <Grid container alignItems='right' justifyContent='right'>
        <TipsIcon
          title={commonTips.CST_TIPS_TITLE_TODAYRECEPTION}
          content={commonTips.CST_TIPS_CONTENT_TODAYRECEPTION}
        >
        </TipsIcon>
      </Grid>
      <Grid>
        <StyledHeader>
          <InputSelect 
            list={completeClassList}
            title="完了区分："
            name="nm1"
            indx="comleteClassName"
            value="cd1"
            column="completeClassName"
            unqkey="cd1"
            defaultValue={conditions.completeClass}
            onChange={handleOnChangeCompleteKbn}
            options={{blank:true,all:false}}
          />
        </StyledHeader>
      </Grid>
      <br></br>
      <Box
        sx = {{
          bgcolor: 'info.main',
          color: 'background.paper',
          p: 2 ,
          textAlign: 'center',
          maxWidth : "100%",
        }}
      >
      <StyledTitleHeader>
        <strong>本日の応対一覧</strong>
      </StyledTitleHeader>
      </Box>
      <Grid>
        <Box
          sx = {{
            height: '90%',
            width: '100%',
          }}
        >
          <DataGrid_list 
            list = {todayReceptionList}
          />
        </Box>
      </Grid>
      <br></br>
      <Box
        sx = {{
          height: '100%',
          width: '100%',
        }}
      >
        <Grid container alignItems='right' justifyContent='right'>
          <Button
            variant="contained"
            color="primary"
            onClick={() => handleIsShowReception(conditions.isShowReception)}
            size='large'
          >
            {receptionButtonText}
          </Button>
        </Grid>
      </Box>
    </React.Fragment>
  )
};

export const Index = ():JSX.Element => {
  return (
    <React.Fragment>
      <Layout.Index mainComponent={<Main />} title='本日応対' />
    </React.Fragment>
    
  );
};

export default Index;
\\\\\\\\\\\\\\\---
------
datagript.tsx
------------
import React from 'react'
import { DataGrid, GridColDef, jaJP } from '@mui/x-data-grid';
import { Button, Link as MuiLink } from '@mui/material';
import Box from '@mui/material/Box';
import * as common from '../../constants/Common';
import NextLink from "next/link";
import { StyledDataGrid } from "../../styles/Styled";

const DataGrid_list = (props) => {
  const [pageSize, setPageSize] = React.useState<number>(10);

  // 改行コード置換（\r\nを<BR/>に）
  const replaceContent = (content) => {
    const str = content.split(/(\r\n)/).map((item,index) => {
        return (
            <React.Fragment key={index}>
                { item.match(/\r\n/) ? <br /> : item }
            </React.Fragment>
        );
    });
  
    return <div>{str}</div>
  }
  
  const columns: GridColDef[] = [
    { field: 'rowNo', headerName: 'No.', type:'string', width: 70 , headerAlign: 'center', align: 'center', flex: 0.2},
    {
      field: 'editBtn',
      headerName: '',
      align: 'center',
      flex: 0.2,
      disableClickEventBulling:true,
      renderCell: (params) => {
        if(!params.row.datFlg) {
          return  <NextLink
                    href={{
                      pathname:"BasicRegist",
                      query:{flg: "update", customerCd: params.row.customerCd, respondNo : params.row.respondNo}
                    }} as = "BasicRegist"
                  >
                    <Button
                      variant="contained"
                      color="primary"
                      size="small"
                    >詳細</Button>
                  </NextLink>
        } else {
          return  <Button
                    disabled={params.row.datFlg}
                    variant="contained"
                    color="primary"
                    size="small"
                  >詳細</Button>
        }
    }} as GridColDef,
    { field: 'customerNm', 
      headerName: '顧客名', 
      type:'string', 
      headerAlign: 'center',
      align: 'left',
      flex: 0.5, 
      renderCell: (params) => {
        if(!params.row.datFlg) {
          return <NextLink
                  href={{
                    pathname:"BasicRegist",
                    query:{customerCd : params.row.customerCd}
                  }} passHref as = "BasicRegist"
                 >
                 <MuiLink>{params.row.customerNm}</MuiLink>
                 </NextLink>
        } else {
          return params.row.customerNm
        }
    }},
    { field: 'receptionTm', headerName: '時間', type:'string', headerAlign: 'center', align: 'center', flex: 0.2 },
    { field: 'content', headerName: '応対内容', type:'string', headerAlign: 'center', align: 'left', flex: 0.5 },
    { field: 'customerRequire',
      headerName: '顧客用件',
      type:'string',
      headerAlign: 'center',
      align: 'left',
      flex: 0.8,
      renderCell:(params) => {
        return replaceContent(params.row.customerRequire)
    }},
    { field: 'cfsContent',
      headerName: 'CFS対応',
      type:'string',
      headerAlign: 'center',
      align: 'left',
      flex: 0.8,
      renderCell:(params) => {
        return replaceContent(params.row.cfsContent)
    }},
    { field: 'complete', headerName: '完了', type:'string', headerAlign: 'center', align: 'center', flex: 0.2 },
    { field: 'operatorNm', headerName: '担当者', type:'string', headerAlign: 'center', align: 'left', flex: 0.5 },
    // 隠し項目
    { field: 'customerCd', headerName: '顧客コード', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'respondNo', headerName: '応対No', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'completeClass', headerName: '完了分類', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'completeFlg', headerName: '完了フラグ', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'deliveryFlg', headerName: '留守フラグ', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'datFlg', headerName: '論理削除フラグ', type:'string', headerAlign: 'center',align: 'left', hide: true },
  ];

  return (
    <Box
      sx={{
        height: '100%',
        width: '100%',
        // 完了区分（COMPLETEFLG）が「true：完了」は背景色を変更
        "& .super-app-theme--COMP": {
          bgcolor:common.CST_COLOR_GREEN,
          "&:hover": {
            bgcolor:common.CST_COLOR_HB_GREEN
          }
        },
        // 留守コード（DELIVERYFLG）が「true：留守」はフォントカラーを変更
        "& .super-app-theme--DELIVERY": {
          color:common.CST_COLOR_RED,
          fontWeight:'600',
        }
    }}
    >
      <div>
        <DataGrid getRowId={(row) => row.rowNo}
          sx={StyledDataGrid.grid}
          rows={props.list.map((row) => {
            return {
              ...row,
              rowNo:props.list.indexOf(row)+1,
            }
          })}
//          rows={props.list}
          getRowHeight={() => 'auto'}
          columns={columns}
          pageSize={pageSize}
          onPageSizeChange={(newPageSize) => setPageSize(newPageSize)}
          rowsPerPageOptions={[10, 15, 30, 50]}
          pagination
          getCellClassName={(params) => 
            {if(params.field !== "complete") {
              return "";
            }
            if(!params.row.deliveryFlg){
              return "";
            }
            return `super-app-theme--DELIVERY`}
          }
          getRowClassName={(params) => 
            {if(!params.row.completeFlg) {
              return "";
            }
            return `super-app-theme--COMP`}
          }
          localeText={jaJP.components.MuiDataGrid.defaultProps.localeText}
          disableColumnMenu                     // グリッド内のカラムメニュー無効
          disableSelectionOnClick               // 行選択無効
          density='compact'
        />
      </div>
    </Box>
  )
};

export default DataGrid_list;
------------------------

todaystyle
------
import { styled } from '@mui/material/styles';

// ヘッダ部エリアのスタイル設定
//  （検索条件のエリア）
export const StyledHeader = styled("div")(({theme}) => ({
    padding:0,
    paddingLeft: theme.spacing(2),
    paddingTop: '10px',
    fontSize: 24,
  }));

// 一覧画面用ヘッダタイトルのスタイル設定
export const StyledTitleHeader = styled("div")({
  fontSize: 20,
});

// 入力項目のサンプル表記のスタイル設定
export const StyledExample = styled('text')({
  color:'blue',
});

export const StyledDataGrid = {
  grid: {
    '.MuiDataGrid-toolbarContainer': {
      borderBottom: 'solid 1px rgba(224, 224, 224, 1)'  ,

   
    },
     // 列ヘッダに背景色を指定
    '.MuiDataGrid-columnHeaders': {
      backgroundColor: '#40b4c8', 
      color: '#fff',
      fontSize: '1.2rem'
    },
    '.MuiDataGrid-row': {
      fontSize: '1.1rem'
    },
     //　ヘッダ選択時のフォーカス枠の切れを修正
    '.MuiDataGrid-columnHeader:focus-within': {
      outlineOffset: -3,
      outline: 'none'
    },
    //　データ選択時のフォーカス枠の切れを修正
    '.MuiDataGrid-cell:focus': {
      outline: 'none'
    },
    height:592,
    overflowX: 'auto',
  
  },
};

----select
-----
import React, { useEffect, useState } from 'react';
import { styled } from '@mui/system';
import { FormControl, InputLabel, NativeSelect, Select, MenuItem  } from '@mui/material';
import { SelectProps } from './Type';

const StyleFrom = styled("form") ({
  minWidth: 130
});

export const InputSelect = <T extends {[key:string]:string| number | null}>(props:SelectProps<T>):JSX.Element => {
  const {list,title,indx,column,name,value,unqkey,defaultValue,onChange,options} = props;
  const [selectedValue,setSelectedValue] = React.useState(defaultValue)
  const nonSelected = typeof(defaultValue) === 'string'? '' : 0

  const handleOnChange = (rowData:string) => {
    setSelectedValue(rowData)
    if(list.length === 0) { return }
    console.log(props)
    onChange(indx,column,typeof(defaultValue) === 'string'? rowData : Number(rowData))
  }

  return(
    <FormControl>
      <InputLabel shrink>{title}</InputLabel>
        <StyleFrom>
          <NativeSelect 
            required
            inputProps={{
              name: 'kbn',
              id: 'uncontrolled-native',
            }}
            onChange={e => {handleOnChange(e.target.value as string)}}
            value={String(selectedValue)}
          >
            {options.blank? <option key={0} value={nonSelected}>{''}</option> : ''}
            {
              list?
                list.map((row) => {
                  return(
                    <option key={row[unqkey]} value={String(row[value])}>{row[name]}</option>)
                })
              : <option key={-1} value={0}>{'読込失敗'}</option>
            }
            {options.all? <option key={99} value={nonSelected}>全て</option> : ''}
          </NativeSelect >
        </StyleFrom>
      </FormControl>
  )
};

----------------type seclect
---
export interface Option  {
  blank: boolean,
  all: boolean
};

export interface SelectProps<T extends {[key:string]:string|number|null}> {
  list:T[],
  title:string,
  indx:string,
  column:string,
  name:string,
  value:string,
  unqkey:string,
  defaultValue:string | number,
  onChange:(key:string,column:string,value:string | number) => void,
  options:Option
};
----operation today
----
import axios from 'axios';
import { createAsyncThunk } from '@reduxjs/toolkit'
import { TodayReceptionConditions, TodayReceptionDetail } from "./Type";

const url = String(process.env.NEXT_PUBLIC_API_URL);

export const fetchAsyncTodayReceptionList = createAsyncThunk<TodayReceptionDetail[],TodayReceptionConditions>(
  "todayReception/detail",
  async (conditions:TodayReceptionConditions) => {
    // 担当者コード、表示対象の区分（受付分全て表示：true、担当顧客分のみ表示：false）

    // TODO 担当者コードはログイン担当者コードを設定（セッション？）
    const operatorId = '10000001';

    const res = await axios.get<TodayReceptionDetail[]>(`${url}todayReception/detail?operatorid=${operatorId}&mode=${conditions.isShowReception}`)
      .catch(err => {
        return err.response
      });
    if (res.status !== 200) {
      console.log("例外発生時の処理");
      alert('データを取得できませんでした。')
    } else {

      return res.data;
      
    }
  },
);
-----------------
selector today
-----
import { useSelector } from 'react-redux';
import { RootState } from "../../stores/Store";
import { TodayReceptionConditions,TodayReceptionDetail } from './Type';
import { Filter } from "../../helper/Filter";
import { FilterParameter  } from '../../helper/Type';

// 条件を取得
export function useTodayReceptionConditions():TodayReceptionConditions {
  return useSelector((state:RootState) => state.todayReception.conditions);
}

// 明細を取得
export function useTodayReceptionList():TodayReceptionDetail[] {
// テストデータ
//  const list = [
//    {rowNo:"1",customerName:"テスト太郎",receptionTime:"10:15",content:"注文",customerRequire:"用件はありません。\r\nやっぱ用件ありました。確認したところ用件は１件です。\r\nでも用件やっぱりないです。",cfsContent:"CFS対応",complete:"未",operator:"担当者１",customerCode:"000304406431",respondNo:"000100303601",completeClass:"00001",completeKbn:"00001",deliveryKbn:"00001",datKbn:"1"},
//  ];

  const conditions = useSelector((state:RootState) => state.todayReception.conditions);
  const list = useSelector((state:RootState) => state.todayReception.list);

  // filter実行
  var filters:FilterParameter<any>[] = []

  if(conditions.completeClass !== ""){
    filters = filters.concat({key:'completeClass',column:'completeClass',parameter:[conditions.completeClass],comparisonFormula:Filter.equal })
  }

  return Filter.execute(list,filters);
}
-----slice today
----
import { createSlice,PayloadAction } from '@reduxjs/toolkit';
import { TodayReceptionConditions,TodayReceptionDetail,TodayReceptionState} from './Type';
import { fetchAsyncTodayReceptionList } from './Operation';

const initialTodayReceptionState:TodayReceptionState= {
  list:[] as TodayReceptionDetail[],
  conditions:{
    completeClass:'',
    isShowReception:false,
    title:'todayReception',
  } as TodayReceptionConditions,
}

// TodayRecptionList情報のスライス
const todayReceptionSlice = createSlice({
  name: 'todayReception',
  initialState:initialTodayReceptionState,
  reducers: {
    // レデューサーとアクションを生成
     conditionsChanged(state:TodayReceptionState,action:PayloadAction<TodayReceptionConditions>) {
       state.conditions = action.payload
      }
    },
    extraReducers: (builder) => {
      builder.addCase(
        fetchAsyncTodayReceptionList.fulfilled,
        (state:TodayReceptionState, action: PayloadAction<TodayReceptionDetail[]>) => {
          return{
            ...state,
            list: action.payload
          }
        }
      )
    }
});

export default todayReceptionSlice;
----type today
----

export interface TodayReceptionConditions{
  [key:string]:string | boolean,
  title:string,
  completeClass:string,
  isShowReception:boolean
}

export interface TodayReceptionDetail{
  [key:string]:string | number | boolean | null,
  rowNo:string,
  customerCd:string,
  customerNm:string,
  receptionTm:string,
  content:string,
  customerRequire:string,
  cfsContent:string,
  complete:string,
  operator:string,
  completeFlg:boolean,
  deliveryFlg:boolean,
  completeClass:string,
  respondNo:string,
  datFlg:boolean
}

export interface TodayReceptionState{
  conditions:TodayReceptionConditions,
  list:TodayReceptionDetail[]
}

----layout menu
----
import * as MenuLayout from './Layout/index';

import TodayIcon from '@mui/icons-material/Today';
import CallTwoToneIcon from '@mui/icons-material/CallTwoTone';
import DescriptionIcon from '@mui/icons-material/Description';
import HistoryToggleOfIcon from '@mui/icons-material/HistoryToggleOff';
import PersonAddIcon from '@mui/icons-material/PersonAdd';
import MessageIcon from '@mui/icons-material/Message';
import AddShoppingCartIcon from '@mui/icons-material/AddShoppingCart';
import MarkAsUnreadIcon from '@mui/icons-material/MarkAsUnread';
import PersonSearchIcon from '@mui/icons-material/PersonSearch';
import ProductionQuantityLimitsTwoToneIcon from '@mui/icons-material/ProductionQuantityLimitsTwoTone';
import PlaylistAddCheckTwoToneIcon from '@mui/icons-material/PlaylistAddCheckCircleTwoTone';
import CalenderMonthIcon from '@mui/icons-material/CalendarMonth';
import FeaturePlayListIcon from '@mui/icons-material/FeaturedPlayList';
import ForumIcon from '@mui/icons-material/Forum';
import ContactPhoneIcon from '@mui/icons-material/ContactPhone';
import PhoneCallbackIcon from '@mui/icons-material/PhoneCallback';

const MenuCallCenterArr: MenuLayout.MenuItem[] = [
    { id: 1, url:"/CallCenter/TodoList", icon:<TodayIcon />, text:'本日予定'},
    { id: 2, url:"/CallCenter/OutCall", icon:<CallTwoToneIcon />, text:'アウトコール'},
    { id: 9, url:"/CallCenter/Customer", icon:<PersonSearchIcon />, text:'顧客検索'},
    { id: 10, url:"/CallCenter/OrderTodayList", icon:<ProductionQuantityLimitsTwoToneIcon />, text:'受注速報'},
    { id: 11, url:"/CallCenter/RequestState", icon:<PlaylistAddCheckTwoToneIcon />, text:'依頼状況'},
    { id: 12, url:"/CallCenter/TodayReception", icon:<FeaturePlayListIcon />, text:'本日応対'},
    { id: 13, url:"/CallCenter/MsgList", icon:<ForumIcon />, text:'伝言状況'},
    { id: 14, url:"/CallCenter/CallList", icon:<ContactPhoneIcon />, text:'コールリスト'},
    { id: 15, url:"/CallCenter/CallBack", icon:<PhoneCallbackIcon />, text:'折り返し'},
    { id: 16, url:"/CallCenter/ShiftWork", icon:<CalenderMonthIcon />, text:'シフト'},
];

const MenuCallCenterReceptionArr: MenuLayout.MenuItem[] = [
    { id: 3, url:"/CallCenter/BasicRegist", icon:<PersonAddIcon />, text:'応対入力'},
    { id: 4, url:"/CallCenter/Order", icon:<AddShoppingCartIcon />, text:'受注入力'},
    { id: 5, url:"/CallCenter/PurchaseHistory", icon:<HistoryToggleOfIcon />, text:'購入履歴'},
    { id: 6, url:"/CallCenter/SendDoc", icon:<DescriptionIcon />, text:'資料送付'},
    { id: 7, url:"/CallCenter/InputMsg", icon:<MessageIcon />, text:'伝言入力'},
    { id: 8, url:"/CallCenter/DmSendHistory", icon:<MarkAsUnreadIcon />, text:'ＤＭ発送履歴'}
];

export const MenuItems = () =>{ 
    return MenuCallCenterArr.concat(MenuCallCenterReceptionArr)
};

export default MenuItems;

----emplouee
\\\^-----
import  * as React from 'react';
import * as MenuLayout from './Layout/index';

// アイコン関連
import HistoryToggleOfIcon from '@mui/icons-material/HistoryToggleOff';
import AddShoppingCartIcon from '@mui/icons-material/AddShoppingCart';

// #region メニュー配列設定
const MenuEmployeeArr: MenuLayout.MenuItem[] = [
    { id: 1, url:"/CallCenter/Employee/Order", icon:<AddShoppingCartIcon />, text:'社販受注入力'},
    { id: 2, url:"/CallCenter/Employee/OrderHistory", icon:<HistoryToggleOfIcon />, text:'社販購入履歴'}
];
// #endregion

export default function MenuEmployee () {
    return (
        <MenuLayout.Menu MenuArr={MenuEmployeeArr}/>
    );
};
\\\\---------layout
\\\\----
import  * as React from 'react';
import MenuList from '../../components/Menu/Atoms/MenuList';
import  { useState } from 'react';
import { styled, useTheme, Theme, CSSObject } from '@mui/material/styles';
import Box from '@mui/material/Box';
import MuiDrawer from '@mui/material/Drawer';
import CssBaseline from '@mui/material/CssBaseline';
import MuiAppbar, { AppBarProps as MuiAppBarProps } from '@mui/material/AppBar';
import Toolbar from '@mui/material/Toolbar';
import Divider from '@mui/material/Divider';
import IconButton from '@mui/material/IconButton';
import clsx from 'clsx';

// アイコン関連
import MenuIcon from '@mui/icons-material/Menu';
import ChevronLeftIcon from '@mui/icons-material/ChevronLeft';
import MenuItems from '../../components/Menu/CallCenter';

const backgroundColor = "##FFFFFF";

interface ComponentProps {
  mainComponent:JSX.Element,
  title:string,
};

const drawerWidth = 240;

// #region メニューのオープン／クローズ設定
const openedMixin = (theme: Theme): CSSObject => ({
    width: drawerWidth,
    transition: theme.transitions.create('width', {
        easing: theme.transitions.easing.sharp,
        duration: theme.transitions.duration.enteringScreen,
    }),
    overflowX: 'hidden',
});

const closedMixin = (theme: Theme): CSSObject => ({
    transition: theme.transitions.create('width', {
        easing: theme.transitions.easing.sharp,
        duration: theme.transitions.duration.leavingScreen,
    }),
    overflowX: 'hidden',
    width: `calc(${theme.spacing(7)} + 1px )`,
    [theme.breakpoints.up('sm')]: {
        width: `calc(${theme.spacing(8)} * 1px )`,
    },
});

interface AppBarProps extends MuiAppBarProps {
    open?: boolean;
};

const AppBar = styled(MuiAppbar, {
    shouldForwardProp: (prop) => prop !== 'open',
})<AppBarProps>(({ theme, open }) => ({
    zIndex: theme.zIndex.drawer + 1,
    transition: theme.transitions.create(['width', 'margin'], {
        easing: theme.transitions.easing.sharp,
        duration: theme.transitions.duration.leavingScreen,
    }),
    ...open && {
        marginLeft: drawerWidth,
        width: `calc(100% - ${drawerWidth}px)`,
        transition: theme.transitions.create(['width', 'margin'], {
            easing: theme.transitions.easing.sharp,
            duration: theme.transitions.duration.enteringScreen,
        }),
    },
}));

const DrawerHeader = styled('div')(({theme}) => ({
    display: 'flex',
    alignItems: 'center',
    justifyContent: 'flex-end',
    padding: theme.spacing(0,1),
    ...theme.mixins.toolbar,
}));

const Drawer = styled(MuiDrawer, { shouldForwardProp: (prop) => prop !== 'open'}) (
    ({ theme, open }) => ({
        width: drawerWidth,
        flexShrink: 0,
        whiteSpace: 'nowrap',
        boxSizing: 'border-box',
        ...(open && {
            ...openedMixin(theme),
            '& .MuiDrawer-paper': openedMixin(theme),
        }),
        ...(!open && {
            ...closedMixin(theme),
            '& .MuiDrawer-paper': closedMixin(theme),
        }),
    }),  
);
// #endregion


export interface MenuItem {
    id: number;
    url: string;
    icon: JSX.Element;
    text: string;
};

export interface MenuProps {
    items: MenuItem[]
};

const StyleForm = styled('form')(({theme}) => ({
    paddingTop:theme.spacing(0),
    backgroundColor:backgroundColor,
  }));
  
  const StyleContent = styled("div")({
    width:'100%',
    overflowX: 'hidden',
  });
  

export const Index = (mainComponentProps:ComponentProps) => {
    // サイドメニューの開閉状態
    const theme = useTheme();
    const [open, setOpen] = useState(true);
    const{ mainComponent,title } = mainComponentProps;

    // オープンクリック
    const handleDrawerOpen = () => {
        setOpen(true);
    };

    // クローズクリック
    const handleDrawerClose = () => {
        setOpen(false);
    };

    return (
      <React.Fragment>
            <Box
                sx = {{ display: 'flex' }}
            >
                <CssBaseline />
                <AppBar
                    position = 'fixed'
                    open={open}
                >
                    <Toolbar>
                        <IconButton
                            edge="start"
                            color='inherit'
                            onClick = {handleDrawerOpen}
                            sx = {{
                                mr: 2,
                                ...(open && { display: 'none'}),
                            }}
                        >
                            <MenuIcon/>
                        </IconButton>
                    </Toolbar>
                </AppBar>
                <Drawer
                    variant = "permanent"
                    open={open}
                    sx = {{
                        '& .MuiDrawer-paper' : {
                            backgroundColor: '#FFFFCC',
                            boxSizing: 'border-box',
                        },
                    }}
                >
                    <DrawerHeader>
                        <IconButton
                            onClick = {handleDrawerClose}
                            sx = {{
                                color: '#000000',
                            }}
                        >
                            {<ChevronLeftIcon/>}
                        </IconButton>
                    </DrawerHeader>
                    
                    {MenuItems().map((menu) => {
                        return (
                            <MenuList key={menu.id} url={menu.url} icon={menu.icon} text={menu.text} color='#87CEFA'/>
                        )
                    })}

                    <Divider />
                </Drawer>
                <Box component='main' sx ={{flexGrow: 1, p: 3}}>
                    <StyleForm>
                        <main className={clsx(StyleContent)}>
                            {mainComponent}
                        </main>
                    </StyleForm>
                </Box>
            </Box>
        </React.Fragment>
    );
};

export default Index;
------commontips^-^^^
// **********************************
// Tips用タイトル　定義
// **********************************
// コールセンター一般
export const CST_TIPS_TITLE_TODOLIST = "本日の予定";
export const CST_TIPS_TITLE_OUTCALL = "アウトコール";
export const CST_TIPS_TITLE_CUSTOMER = "顧客検索";
export const CST_TIPS_TITLE_ORDERTODAYLIST = "受注速報";
export const CST_TIPS_TITLE_REQUESTSTATE = "依頼状況";
export const CST_TIPS_TITLE_TODAYRECEPTION = "本日応対";
export const CST_TIPS_TITLE_MSGLIST = "伝言状況";
export const CST_TIPS_TITLE_CALLLIST = "コールリスト";
export const CST_TIPS_TITLE_CALLBACK = "折り返し";
export const CST_TIPS_TITLE_SHIFTWORK = "シフト";
export const CST_TIPS_TITLE_BASICREGIST = "応対入力";
export const CST_TIPS_TITLE_ORDER = "受注入力";
export const CST_TIPS_TITLE_PURCHASEHISTORY = "購入履歴";
export const CST_TIPS_TITLE_SENDDOC = "資料送付";
export const CST_TIPS_TITLE_INPUTMSG = "伝言入力";
export const CST_TIPS_TITLE_DMSENDHISTORY = "DM発送履歴";

// 社販受注
export const CST_TIPS_TITLE_EMPLOYEE_ORDER = "社販受注";
export const CST_TIPS_TITLE_EMPLOYEE_PURCHASEHISTORY = "社販購入履歴";

// コールセンター管理
export const CST_TIPS_TITLE_REGISTOPERATOR = "担当者登録";
export const CST_TIPS_TITLE_SETAUTHORITY = "セキュリティ設定";
export const CST_TIPS_TITLE_INPUTANNOUNCE = "メッセージ入力";
export const CST_TIPS_TITLE_SALESTARGET = "売上目標";
export const CST_TIPS_TITLE_SHIPMANAGEMENT = "出荷管理";

// 社販受注管理
export const CST_TIPS_TITLE_EMPLOYEE_CUSTOMERMAINTE = "社販顧客メンテ";
export const CST_TIPS_TITLE_EMPLOYEE_CHANGECUSTOMER = "社販顧客変更";
export const CST_TIPS_TITLE_EMPLOYEE_SCHEDULE = "社販スケジュール";

// オファー管理
export const CST_TIPS_TITLE_OFFER = "オファー管理";

// 在庫管理
export const CST_TIPS_TITLE_STOCKMANAGE = "在庫管理";

// 受注取込
export const CST_TIPS_TITLE_IMPORTORDER = "受注取込";

// EC伝票取込
export const CST_TIPS_TITLE_IMPORTECORDER = "EC伝票取込";

// **********************************
// Tips用本文　定義
// **********************************
// コールセンター一般
export const CST_TIPS_CONTENT_TODOLIST = "本日の予定TIPS";
export const CST_TIPS_CONTENT_OUTCALL = "アウトコールTIPS";
export const CST_TIPS_CONTENT_CUSTOMER = "顧客検索TIPS";
export const CST_TIPS_CONTENT_ORDERTODAYLIST = "受注速報TIPS";
export const CST_TIPS_CONTENT_REQUESTSTATE = "依頼状況TIPS";
export const CST_TIPS_CONTENT_TODAYRECEPTION = "●一覧の背景色・文字色について\n　背景色が緑：完了している応対情報になります。\n　文字色が赤：留守区分が1の応対情報になります。\n\n" +
                                               "●担当顧客分のみ表示ボタン押下時\n　ログイン者の担当顧客分の応対情報を表示します。\n　（ボタン名が「受付分全て表示」に変わります）\n" +
                                               "●受付分全て表示ボタン押下時\n　担当でない顧客の応対情報も表示します。\n　（ボタン名が「担当顧客分のみ表示」に変わります）\n\n" +
                                               "●完了区分のリストボックス選択時\n　選択された完了区分に合致する応対情報の一覧が\n　表示されます。\n\n" +
                                               "●詳細ボタン押下時\n　該当顧客の新規応対登録画面が表示されます。\n\n" +
                                               "●顧客名リンク押下時\n　該当顧客の応対入力画面が表示されます。\n　（ログイン者の権限がないもしくは、抹消顧客の場合は\n　　リンク表示されません）"
                                               ;
export const CST_TIPS_CONTENT_MSGLIST = "●一覧の背景色・文字色について\n　完了しているデータは背景色をグレーとしており、\n　状態欄に「完了」が表示されます。\n\n" +
                                        "●コール区分リストボックス選択時（業務部門のみ表示）\n　選択されたコール区分に合致する伝言情報の一覧が\n　表示されます。\n　未選択時は以下のコール区分の情報が表示されます。" +
                                        "\n　　・伝言　　　・返品理由保留\n　　・返品確認　・調査依頼\n\n" +
                                        "●顧客名リンク押下時\n　該当顧客の応対入力画面が表示されます。\n　（ログイン者の権限がないもしくは、抹消顧客の場合は\n　　リンク表示されません）\n\n" +
                                        "●自動送信分を出すボタン押下時（業務部門のみ表示）\n　システムより自動送信された伝言情報を表示します。\n　（ボタン名が「本人送信分を出す」に変わります）\n\n" +
                                        "●本人送信分を出すボタン押下時\n　本人が送信した伝言情報を表示します。\n　（ボタン名が「自動送信分を出す」に変わります）\n\n" +
                                        "●完了分を出す（出さない）ボタン押下時\n　コール結果が完了している情報も一覧に表示します。\n　（完了分を出さない押下時はコール結果が完了している\n　　情報は一覧に表示されません）\n\n" +
                                        "●移動ボタン押下時\n　指定された日付の伝言情報を表示します。\n　本日以外を指定した場合は「完了分を出す」設定となり、\n　完了分を出す（出さない）ボタンは無効になります。"
                                        ;
export const CST_TIPS_CONTENT_CALLLIST = "コールリストTIPS";
export const CST_TIPS_CONTENT_CALLBACK = "折り返しTIPS";
export const CST_TIPS_CONTENT_SHIFTWORK = "シフトTIPS";
export const CST_TIPS_CONTENT_BASICREGIST = "応対入力TIPS";
export const CST_TIPS_CONTENT_ORDER = "受注入力TIPS";
export const CST_TIPS_CONTENT_PURCHASEHISTORY = "購入履歴TIPS";
export const CST_TIPS_CONTENT_SENDDOC = "資料送付TIPS";
export const CST_TIPS_CONTENT_INPUTMSG = "伝言入力TIPS";
export const CST_TIPS_CONTENT_DMSENDHISTORY = "DM発送履歴TIPS";

// 社販受注
export const CST_TIPS_CONTENT_EMPLOYEE_ORDER = "社販受注TIPS";
export const CST_TIPS_CONTENT_EMPLOYEE_PURCHASEHISTORY = "社販購入履歴TIPS";

// コールセンター管理
export const CST_TIPS_CONTENT_REGISTOPERATOR = "担当者登録TIPS";
export const CST_TIPS_CONTENT_SETAUTHORITY = "セキュリティ設定TIPS";
export const CST_TIPS_CONTENT_INPUTANNOUNCE = "メッセージ入力TIPS";
export const CST_TIPS_CONTENT_SALESTARGET = "売上目標TIPS";
export const CST_TIPS_CONTENT_SHIPMANAGEMENT = "出荷管理TIPS";

// 社販受注管理
export const CST_TIPS_CONTENT_EMPLOYEE_CUSTOMERMAINTE = "社販顧客メンテTIPS";
export const CST_TIPS_CONTENT_EMPLOYEE_CHANGECUSTOMER = "社販顧客変更TIPS";
export const CST_TIPS_CONTENT_EMPLOYEE_SCHEDULE = "社販スケジュールTIPS";

// オファー管理
export const CST_TIPS_CONTENT_OFFER = "オファー管理TIPS";

// 在庫管理
export const CST_TIPS_CONTENT_STOCKMANAGE = "在庫管理TIPS";

// 受注取込
export const CST_TIPS_CONTENT_IMPORTORDER = "受注取込TIPS";

// EC伝票取込
export const CST_TIPS_CONTENT_IMPORTECORDER = "EC伝票取込TIPS";


------------store

import { configureStore, combineReducers,ThunkAction,Action } from '@reduxjs/toolkit';
import logger from 'redux-logger';
import todoSlice from '../features/TodoList/Slice';
import callClassSlice from '../features/CallClass/Slice';
import operatorReducer from '../features/Operator/Slice';
import prefectureReducer from '../features/Prefecture/Slice';
import completeClassSlice from '../features/CompleteClass/Slice';
import todayReceptionSlice from '../features/TodayReception/Slice';
import msgListSlice from '../features/MsgList/Slice';
import msgKbnReducer from '../features/InputMsg/Slice';

const NODE_ENV = process.env.NODE_ENV;

const rootReducer = combineReducers({
  todo: todoSlice.reducer,
  callClass:callClassSlice.reducer,
  //operator:operatorSlice.reducer,
  operator:operatorReducer,
  prefecture:prefectureReducer,
  completeClass:completeClassSlice.reducer,
  todayReception:todayReceptionSlice.reducer,
  msgList:msgListSlice.reducer,
  msgKbn:msgKbnReducer,
})

export const store = configureStore({
    reducer: rootReducer,
    //middleware: (NODE_ENV === 'production')? [ ...getDefaultMiddleware({...getDefaultMiddleware,serializableCheck: false})] : [ ...getDefaultMiddleware({...getDefaultMiddleware,serializableCheck: false}), logger]
    middleware: (NODE_ENV === 'production')? 
                 (getDefaultMiddleware) => getDefaultMiddleware({
                  serializableCheck: false,
                 }).concat(logger)
              : (NODE_ENV === 'development')?
                 (getDefaultMiddleware) => getDefaultMiddleware({
                  serializableCheck: false,
                 }).concat(logger)
              : (getDefaultMiddleware) => getDefaultMiddleware({
                serializableCheck: false,
                }).concat(logger),
})

export type RootState = ReturnType<typeof rootReducer>;
export type AppDispatch = typeof store.dispatch;
export type AppThunk<ReturnType = void> = ThunkAction<
  ReturnType,
  RootState,
  unknown,
  Action<string>
>;
-------------------------------
tips icon common
----
import InfoIcon from '@mui/icons-material/InfoRounded';
import { Tooltip, Typography } from '@mui/material';
import IconButton from "@mui/material/IconButton";
import {styled as iconstyled} from "@mui/styles";
import Modal from "@mui/material/Modal";
import React from 'react';
import Box from '@mui/system/Box';


// tips用アイコンスタイル設定
const IconStyle = iconstyled(InfoIcon)({
    width:35,
    height:35,
 });

// モーダルスタイル設定
const ModalStyle = {
    position: 'absolute',
    top: '10%',
    right: 100,
    width: 550,
    maxHeight:700,
    bgcolor: 'background.paper',
    border: '2px solid #000',
    boxShadow: 24,
    p: 4,
    overflow:"scroll"
}

// 改行コード置換（\nを<BR/>に）
const replaceContent = (content) => {
    const str = content.split(/(\n)/).map((item,index) => {
        return (
            <React.Fragment key={index}>
                { item.match(/\n/) ? <br /> : item }
            </React.Fragment>
        );
    });

    return <div>{str}</div>
}

export default function TipsIcon(props) {
    const [open, setOpen] = React.useState(false);
    const handleOpen = () => setOpen(true);
    const handleClose =() => setOpen(false);
    
    return (
        <React.Fragment>
            <Tooltip title="Tips">
               <IconButton
                    size = "small"
                    color = "warning"
                    onClick = {handleOpen}
                    sx = {{marginRight:'10px'}}
                >
                    <IconStyle>
                        <InfoIcon/>
                    </IconStyle>          
                </IconButton>
            </Tooltip>

            <div>
                <Modal
                    open={open}
                    onClose={handleClose}
                    aria-labelledby="modal-modal-title"
                    aria-descrobedby="modal-modal-description"
                >
                <Box sx={ModalStyle}>
                    <Typography id="modal-modal-title" variant="h6" component="h2">
                        <b>{props.title}のTips！</b>
                    </Typography>
                    <Typography id="modal-modal-description" sx={{ mt:2 }}>
                        {replaceContent(props.content)}
                    </Typography>
                </Box>
                </Modal>
            </div>
        </React.Fragment>
    );    
}
