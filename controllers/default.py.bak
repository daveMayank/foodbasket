# -*- coding: utf-8 -*-
# this file is released under public domain and you can use without limitations

#########################################################################
## This is a sample controller
## - index is the default action of any application
## - user is required for authentication and authorization
## - download is for downloading files uploaded in the db (does streaming)
#########################################################################

def index():
    """
    example action using the internationalization operator T and flash
    rendered by views/default/index.html or views/generic.html

    if you need a simple wiki simply replace the two lines below with:
    return auth.wiki()
    """
    if not auth.is_logged_in():
        return {'message':"This is not logged in page"}
    else:
        if(auth.user.userType == 'Customer'):
            redirect(URL('signed'))
        elif( auth.user.userType == 'Restaurant'):
            redirect(URL('manage_restau'))
        else :
            redirect(URL('myadminHome'))#change to appadmin, how ?

def helpFunction():
    return {'message':"Thisis help page"}

@auth.requires_login()
def myadminHome():
    if (auth.has_membership(18)):
        redirect(URL('signed'))
    else:
        records = db(db.temporary_restaurant.id > 0).select()
        return locals()

@auth.requires_login()
def mainAdminRedirect():
    redirect(URL('../appadmin'))
    
@auth.requires_login()
def approve_restau():
    tid = request.vars.restid
    records = db(db.temporary_restaurant.id == tid).select()
    resname = records[0].rname
    resloc  = records[0].locality
    resInfo = records[0].info
    resRating = records[0].rating
    resImage = records[0].hotelImage
    resown   = records[0].rest_owner
    db.restaurant.insert(rname=resname, locality=resloc, info=resInfo, rating=resRating, hotelImage=resImage, rest_owner=resown)
    db(db.temporary_restaurant.id == tid).delete()
    redirect(URL('myadminHome'))
    
@auth.requires_login()
def manage_restau():
    userType=auth.user.userType
    if (userType == 'Restaurant'):
        records = db(db['restaurant']['rest_owner'] == auth.user.id).select()
    else:
        pass
        #redirect
    return locals()

@auth.requires_login()
def add_restau(): #how to check for membership/type
    form = SQLFORM(db.temporary_restaurant)
    if form.process().accepted:
        session.flash = "Kindly wait for approval"
        redirect(URL('manage_restau'))
    elif form.errors:
        response.flash = 'form has errors'
    else:
        pass
        #response.flash = 'please fill out the form'
    return dict(form=form)

@auth.requires_login()
def remove_restau():
    rid = request.vars.restid
    db(db.restaurant.id == rid).delete()
    session.flash = "Restaurant removed"
    redirect(URL('manage_restau'))

@auth.requires_login()
def editMenu():
    rid = request.vars.restid
    grid = SQLFORM.grid(db.rmenu.rest_name == rid)
    return locals()
    
@auth.requires_login()
def orderStatus():
    return locals()
    
@auth.requires_login()
def signed():
    if session.cart:
        session.cart=None
    if session.namecart:
        session.namecart=None
    form=FORM('Enter Your Area', DIV(INPUT(_type='text',_name='locality',requires=IS_NOT_EMPTY(),_placeholder="Search for...",_class="form-control"),
             SPAN(INPUT(_type='submit',_class="btn btn-default"),_class="input-group-btn"),_class="input-group"))
    if form.accepts(request,session):
        session.flash = request.vars.locality
        redirect(URL('dispRests',args=[request.vars.locality]))
    elif form.errors:
        response.flash = 'form has errors'
    else:
        pass
    #   response.flash = 'please fill the form'
    return dict(form=form)

@auth.requires_login()
def dispRests():
    if session.cart:
        session.cart=None
    if session.namecart:
        session.namecart=None
    region=request.args(0)
    region=region.lower()
    records = db(db.restaurant.locality.lower().like(region)).select() #db(db.log.event.like('port%')).select()
    #check if records is 0
    lengthVar =1
    if(len(records) == 0):
        suggest = db(db.restaurant).select()
        
        lengthVar = 0
    return locals()
    #return {'message':region}

@auth.requires_login()
def dispMenu():
    if not session.cart:
        session.cart={}
    if not session.namecart:
        session.namecart={}
    name=request.vars.rname
    rows=db(db['restaurant']['rname']==name).select()
    rid=rows[0].id
    categories=db(db['rmenu']['rest_name'] == rid).select(db.rmenu.category,distinct=True)
    #put if condition here to Assert that record size is 1
    records = db(db['rmenu']['rest_name'] == rid).select(orderby=db.rmenu.category)
    return locals()

@auth.requires_login()
def addCart():
    #return {'message':'123'}
    message1=request.vars.itemid
    if(request.vars.actAs == 'add'):
        temp = session.cart.get(request.vars.itemid,0) + 1
        session.cart[request.vars.itemid] = temp
        session.namecart[request.vars.itemid] = request.vars.itemName
        #message2=str("%s" %temp)
    else:
        temp = session.cart.get(request.vars.itemid,0) - 1
        if(temp <= 0):
            temp=0
            del session.cart[request.vars.itemid]
            del session.namecart[request.vars.itemid]
        else:
            session.cart[request.vars.itemid] = temp
        #message2=str("%s" %temp)
    
    #return message2
    testString = "italic"
    temporary_code = ""
    for i in session.cart.keys():
                        if i is not None:
                            temporary_code = temporary_code +   str(A(DIV(_class="glyphicon glyphicon-minus-sign"),_href="#", _onclick="ajax('%s',[],'orderTracker')" %  URL('default','addCart',vars=dict(itemid=i,actAs='sub'))))+ "        " + str(session.cart.get(i,0)) +"        " + str(session.namecart.get(i,0))  + "<br>"

    html_resp = DIV(XML('<p>  <div class="glyphicon glyphicon-shopping-cart"></div>  Your order</p>'),
                    XML(temporary_code)
                     )
    return html_resp

@auth.requires_login()
def dispOrder():
    records = {}
    cost={}
    totalBill=0
    tableString = "<table width='100%' border='1' style='background:#6699FF;'><tr><th>Sr. No</th><th>Item</th><th>Qty</th><th>Price</th></tr>" 
    count = 1;
    for i in session.cart.keys():
        #print i
        if i is not None:
            records[i]=db.rmenu(i)
            cost[i]=records[i].cost_item*session.cart.get(i,0)
            totalBill = totalBill + cost[i]
            tableString = tableString + "<tr><td>%d</td><td>%s</td><td>%d</td><td>%s</td></tr>" % (count, records[i].iname, session.cart.get(i,0), str(records[i].cost_item*session.cart.get(i,0)) )
            count = count + 1
    #test = TABLE(THEAD(TR(TH(), TH(''), TH('Qty'), TH('Price'))),  (TR(TD(1),TD(1),TD(1),TD(1)) for i in range(5)))
    tableString = tableString + "</table>"
    tab = XML(tableString)
    return locals()

from gluon.tools import Mail
mail = Mail()

mail.settings.server = 'smtp.gmail.com:587'
mail.settings.sender = 'gauravcs107@gmail.com'
mail.settings.login = 'gauravcs107@gmail.com:wgbexhqlhyssxvpb'

@auth.requires_login()
def confirm_order():
    orderId1 = str(request.now)+str(auth.user.id)
    #print order_id
    customerId = auth.user.id
    order_time = request.now
    db.orderTable.insert(order_id=orderId1, customer_id=customerId,order_date=order_time)
    orderRow = db(db['orderTable']['order_id'] == orderId1).select()
    oid = orderRow[0].id
    tableString = "<html><table><tr><th>Sr. No</th><th>Item</th><th>Qty</th><th>Price</th></tr>"
    count = 1;
    for i in session.cart.keys():
        #print i
        if i is not None:
            record=db.rmenu(i)
            rid = record.id
            price1=record.cost_item
            qty = session.cart.get(i,0)
            db.orderItem.insert(orderId=oid,orderItemId=rid,quantity=qty,price=price1 )
            tableString = tableString + "<tr><td>%d</td><td>%s</td><td>%d</td><td>%s</td></tr>" % (count, record.iname, session.cart.get(i,0), str(record.cost_item*session.cart.get(i,0)) )
            count = count + 1
    emailId = auth.user.email
    tableString = tableString + "</table></html>"
    x = mail.send(to=[emailId],
    subject='Your order confirmation at FoodBasket',
    message= str(XML(tableString)))
    if x == True:
        response.flash = 'email sent sucessfully.'
    else:
        response.flash = 'fail to send email sorry!'

    session.flash= "Bill has been mailed to you.Expected delivery time is 1 hour"
    session.cart = None
    session.namecart=None
    redirect(URL('signed'))

@auth.requires_login()
def cancel_order():
    session.cart = None
    redirect(URL('signed'))

def user():
    """
    exposes:
    http://..../[app]/default/user/login
    http://..../[app]/default/user/logout
    http://..../[app]/default/user/register
    http://..../[app]/default/user/profile
    http://..../[app]/default/user/retrieve_password
    http://..../[app]/default/user/change_password
    http://..../[app]/default/user/manage_users (requires membership in
    http://..../[app]/default/user/bulk_register
    use @auth.requires_login()
        @auth.requires_membership('group name')
        @auth.requires_permission('read','table name',record_id)
    to decorate functions that need access control
    """
    return dict(form=auth())


@cache.action()
def download():
    """
    allows downloading of uploaded files
    http://..../[app]/default/download/[filename]
    """
    return response.download(request, db)


def call():
    """
    exposes services. for example:
    http://..../[app]/default/call/jsonrpc
    decorate with @services.jsonrpc the functions to expose
    supports xml, json, xmlrpc, jsonrpc, amfrpc, rss, csv
    """
    return service()
