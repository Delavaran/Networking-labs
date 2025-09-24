# Ospf Route Filtering

---

## Ospf filtering overview 

### Ospf is a link state IGP
* routers know the entire topology graph within an area 
* Input to **spf** must be equal in the area to result in the same **SPT**
* Implies that filtering can only be applied at certain points (mostly Abr's)

---

## Ospf route filtering methods :
### Ospf filtering is normally applied between area's 
* Filtering with summarization 
* Lsa Type-3 Filtering (Area Filter_List in Abr)
* NSSA Abr External Prefix Filtering (nssa-only)
* Transit Prefix Suppression(typically for dmvpn or mpls)

### Ospf Rib can be filtered any where 
* Distribute list with Acl or Route-map
* Administrative Distance (To poison a route)

**!** Rib filtering can be dangerous : Does not stop the flooding of the lsa within the area can cause black-hole for traffic 



