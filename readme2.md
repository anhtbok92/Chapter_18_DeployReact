# Xây dựng giỏ hành sử dụng React + Redux

## Cài đặt Project React

```angular2svg
npx create-react-app create-react-redux
npm install axios bootstrap@4.2.1 redux react-redux redux-thunk react-router-dom
```

- API hỗ trợ : https://5adc8779b80f490014fb883a.mockapi.io/products

## Call API

- Create api : src/api/index.js
```angular2svg
import axios from 'axios';
//mock API
let API_URL = 'https://5adc8779b80f490014fb883a.mockapi.io';
   export default function callApi(endpoint, method = 'GET', body) {
       return axios({
           method,
           url: `${API_URL}/${endpoint}`,
           data: body
       }).catch(err => {
           console.log(err);
       });
}

```

- Create action : src/action/index.js

```angular2svg
import callApi from '../api'
export const INCREASE_QUANTITY = 'INCREASE_QUANTITY';
export const DECREASE_QUANTITY = 'DECREASE_QUANTITY';
export const GET_ALL_PRODUCT = 'GET_ALL_PRODUCT';
export const GET_NUMBER_CART = 'GET_NUMBER_CART';
export const ADD_CART = 'ADD_CART' ;
export const UPDATE_CART = 'UPDATE_CART';
export const DELETE_CART = 'DELETE_CART';
 
export const actFetchProductsRequest = () => {
    return (dispatch) => {
        return callApi('/products', 'GET', null).then(res => {
           
            dispatch(GetAllProduct(res.data));
        });
    }
}
 
/* GET_ALL_PRODUCT */
export function GetAllProduct(payload){
    return {
        type:'GET_ALL_PRODUCT',
        payload
    }
}
 
/* GET NUMBER CART */
export function GetNumberCart(){
    return {
        type:'GET_NUMBER_CART'
    }
}
 
export function AddCart(payload){
    return {
        type:'ADD_CART',
        payload
    }
}
export function UpdateCart(payload){
    return {
        type:'UPDATE_CART',
        payload
    }
}
export function DeleteCart(payload){
    return{
        type:'DELETE_CART',
        payload
    }
}
 
export function IncreaseQuantity(payload){
    return{
        type:'INCREASE_QUANTITY',
        payload
    }
}
export function DecreaseQuantity(payload){
    return{
        type:'DECREASE_QUANTITY',
        payload
    }
}
```

- Create reducers : src/reducers/index.js

```angular2svg
import { combineReducers } from 'redux';
import {GET_ALL_PRODUCT,GET_NUMBER_CART,ADD_CART, DECREASE_QUANTITY, INCREASE_QUANTITY, DELETE_CART} from  '../actions';
 
const initProduct = {
    numberCart:0,
    Carts:[],
    _products:[]
}


export const todoProduct(state = initProduct, action) {
    switch(action.type) {
        case GET_ALL_PRODUCT:
        ...
    }

}
const ShopApp = combineReducers({
    _todoProduct:todoProduct
});
export default ShopApp;
```

- Giải thích:

+ Khởi tạo dữ liệu State ban đầu:
```angular2svg
const initProduct = {
    numberCart:0, // lưu số lượng sản phẩm đã mua có trong giỏ hàng
    Carts:[], // tạo mảng giỏ hàng đầu tiền là rỗng
    _products:[] // dùng chứa tất cả sản phầm lấy được từ API
}
```

+ action.type==GET_ALL_PRODUCT

```angular2svg
case GET_ALL_PRODUCT:
   return {
        ...state,
        _products:action.payload
   }
```

+ action.type==ADD_CART
    + Nếu numberCart==0 : nghĩa là chưa có sản phẩm nào trong mảng Carts=> thêm sản phẩm vào mảng Carts
    + Nếu numberCart>0
        + Mảng Carts có sản phẩm=>kiểm tra sản phẩm có mua trùng không, nếu trùng tăng quantity++
        + Nếu sản phẩm mua khác với sản phẩm trong Carts=> thêm sản phẩm đó vào Carts
        + Cuối cùng là tăng numberCart++
    
+ action.type==INCREASE_QUANTITY
    + tăng quantity++ của sản phẩm được chọn
    ```angular2svg
        case INCREASE_QUANTITY:
            state.numberCart++
            state.Carts[action.payload].quantity++;      
            return {
                ...state
            }
    ```
  
+ action.type==DECREASE_QUANTITY
    + giảm quantity-- của sản phẩm được chọn
    ```angular2svg
        case DECREASE_QUANTITY:
            let quantity = state.Carts[action.payload].quantity;
            if(quantity>1){
                state.numberCart--;
                    state.Carts[action.payload].quantity--;
            }         
            return{
                ...state
            }
    ```
  
+ action.type==DELETE_CART
    + xóa sản phẩm được chọn
    ```angular2svg
        case DELETE_CART:
            let quantity_ = state.Carts[action.payload].quantity;
            return {
                ...state,
                numberCart:state.numberCart - quantity_,
                Carts:state.Carts.filter(item=>{
                    return item.id!=state.Carts[action.payload].id
                })        
            }
    ```

- Create store : src/store/index.js

```angular2svg
import { createStore, applyMiddleware } from 'redux';
import thunkMiddleware from 'redux-thunk';
import ShopApp from '../reducers/index'
const store =  createStore(ShopApp,applyMiddleware(thunkMiddleware));
export default store;
```

- Create Header Component

```angular2svg
<li className="nav-item" ><Link to="/" className="nav-link active">Products</Link></li>
<li className="nav-item"><Link to="/carts" className="nav-link">Carts : {this.props.numberCart}</Link></li>
```

- Create Product Component

```angular2svg
const mapStateToProps = state =>{
    return {
         _products: state._todoProduct,
       };
}

function mapDispatchToProps(dispatch){
    return{
        actFetchProductsRequest:()=>dispatch(actFetchProductsRequest()),
        AddCart:item=>dispatch(AddCart(item))
      
    }
}
export default connect(mapStateToProps,mapDispatchToProps)(Product)

```

- Create Card Component

```angular2svg
const mapStateToProps = state =>{
    return{
        items:state._todoProduct
    }
}
 
export default connect(mapStateToProps,{IncreaseQuantity,DecreaseQuantity,DeleteCart})(Cart)

```

- Create App Component 

```angular2svg
import React from 'react';
import {BrowserRouter as Router,Link, Route,Switch} from 'react-router-dom'
import Cart from './components/Cart';
import Header from './components/Header';
import Product from './components/Product';
function App() {
  return (
     <Router>
        <div className="container">
            <Header />
            <Switch>
               <Route path="/" exact component={Product} />
               <Route path="/carts" exact component={Cart} />
            </Switch>
        </div>
     </Router>
  );
}
 
export default App;
```