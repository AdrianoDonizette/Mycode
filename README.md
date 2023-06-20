# Mycode //Um dos códigos que trabalhei

@@ -18,243 +18,195 @@ import com.viridis.domain.infoboard.DynamicReportService;
import com.viridis.txdb.services.GenericDaoService;

public class GabEEInvoiceInformationsReportBDynamicService implements DynamicReportService {
    
    private static final String PARAM_EQUIPMENT_ID = "equipmentId";
    private static final String PARAM_COMPANY_ID = "companyId";
    private static final String PARAM_TO_DATE = "toDate";
    private static final String PARAM_FROM_DATE = "fromDate";
    
    private static final String MISSING_PARAM = "Missing required parameter: ";

    public static final Log LOG = LogFactory.getLog(GabEEInvoiceInformationsReportBDynamicService.class);
    
    @Autowired
    private GenericDaoService genericDaoService;
    
    @Autowired
    private EquipmentDao equipmentDao;
	private static final String PARAM_EQUIPMENT_ID = "equipmentId";
	private static final String PARAM_COMPANY_ID = "companyId";
	private static final String PARAM_TO_DATE = "toDate";
	private static final String PARAM_FROM_DATE = "fromDate";

    @Override
    public Map<String, Object> execute(Map<String, Object> parameters) {
        
         LOG.info("Dynamic Service invoked with parameters: " + parameters);
	private static final String MISSING_PARAM = "Missing required parameter: ";

        Map<String, Object> result = new HashMap<>();
                
        Assert.notNull(parameters.get(PARAM_FROM_DATE), MISSING_PARAM + PARAM_FROM_DATE);
        Assert.notNull(parameters.get(PARAM_TO_DATE), MISSING_PARAM + PARAM_TO_DATE);
	public static final Log LOG = LogFactory.getLog(GabEEInvoiceInformationsReportBDynamicService.class);

        Long fromDate = Long.parseLong(parameters.get(PARAM_FROM_DATE).toString());
        Long toDate = Long.parseLong(parameters.get(PARAM_TO_DATE).toString());
        String companyName = parameters.get(PARAM_COMPANY_ID).toString();
        Long equipmentId = (parameters.get(PARAM_EQUIPMENT_ID) != null) ? Long.parseLong(parameters.get(PARAM_EQUIPMENT_ID).toString()) : null;
        List<Object[]> queryResult = getResult(equipmentId, new Date(toDate), new Date(fromDate), companyName);
      
        
        if (companyName.equals("%") && equipmentId == 0L) {
            throw new DynamicReportServiceException("Não é possível executar o relatório com os dois campos em branco, favor rever os parâmetros selecionados.");
        }
    
        if (queryResult.size() > 4000) {
            throw new DynamicReportServiceException("Foram recuperadas muitas faturas de acordo com o filtro indicado, favor rever os parâmetros selecionados.");
        }
	@Autowired
	private GenericDaoService genericDaoService;

        result.put("result", queryResult);
	@Autowired
	private EquipmentDao equipmentDao;

        return result;
	@Override
	public Map<String, Object> execute(Map<String, Object> parameters) {

    }
    
    @SuppressWarnings("unchecked")
    private List<Object[]> getResult(Long equipmentId, Date toDate, Date fromDate, String companyName) {
        
        String condition = "";
        String fqn = "";
        
        if (equipmentId == 0) {
            if (companyName.equals("Rio Saneamento")) {                
                condition = " AND eq.id in (SELECT eq.id FROM vero_equipment eq, vero_metadata vm where "
                        + " eq.id = vm.entity_id and vm.data -> 'json' ->> 'company' like 'Rio Saneamento') ";
            } else {
                condition = " AND eq.id in (SELECT eq.id FROM vero_equipment eq, vero_metadata vm where "
                        + " eq.id = vm.entity_id and vm.data -> 'json' ->> 'company' like 'GAB') ";
            }
		LOG.info("Dynamic Service invoked with parameters: " + parameters);

        } else {
		Map<String, Object> result = new HashMap<>();

            Equipment equipment = equipmentDao.findOne(equipmentId);
            fqn = equipment.getNamespace() + "." + equipment.getCode() + "%";
            condition = " and eq.namespace || '.' || eq.code like :fqn ";
        }
		Assert.notNull(parameters.get(PARAM_FROM_DATE), MISSING_PARAM + PARAM_FROM_DATE);
		Assert.notNull(parameters.get(PARAM_TO_DATE), MISSING_PARAM + PARAM_TO_DATE);

        String query = ""
                    + " select "
                    + "     eq.name as eqName, c.external_code, "
                    + "     to_char(bd.ref_date, 'MM/YYYY') as ref_date, "
                    + "     to_char(bd.period_from, 'DD/MM/YYYY') as period_from, "
                    + "     to_char(bd.period_to, 'DD/MM/YYYY') as period_to, "
                    + "     bd.number, "
                    + "     cp.name as cpName, "
                    + "     bd.data->'json'->>'TARIFF_CLASS' as CLASSE_TARIFARIA, "
                    + "     case when bd.data->'json'->>'DT_EMISSAO' = '' then null "
                    + "         else to_char(cast(to_timestamp(cast(bd.data->'json'->>'DT_EMISSAO' as bigint)/1000) as date), 'DD/MM/YYYY') "
                    + "     end as DT_EMISSAO, "
                    + "     case when bd.data->'json'->>'DT_VENCIMENTO' = '' then null "
                    + "         else to_char(cast(to_timestamp(cast(bd.data->'json'->>'DT_VENCIMENTO' as bigint)/1000) as date), 'DD/MM/YYYY') "
                    + "     end as DT_VENCIMENTO, "
                    + "     case when (bd.data->'json'->>'C_P' = '') then null "
                    + "         else cast(bd.data->'json'->>'C_P' as float) "
                    + "     end as C_P, "
                    + "     case when (bd.data->'json'->>'C_FP' = '') then null "
                    + "         else cast(bd.data->'json'->>'C_FP' as float) "
                    + "     end as C_FP, "
                    + "     case when (bd.data->'json'->>'C_TOTAL' = '') then null "
                    + "         else cast(bd.data->'json'->>'C_TOTAL' as float) "
                    + "     end as C_TOTAL, "
                    + "     case when (bd.data->'json'->>'D_MEDIDA_P' = '') then null "
                    + "         else cast(bd.data->'json'->>'D_MEDIDA_P' as float) "
                    + "     end as D_MEDIDA_P, "
                    + "     case when (bd.data->'json'->>'D_MEDIDA_FP' = '') then null "
                    + "         else cast(bd.data->'json'->>'D_MEDIDA_FP' as float) "
                    + "     end as D_MEDIDA_FP, "
                    + "     coalesce(case when (bd.data->'json'->>'D_FATURADA_P' = '') then null else cast(bd.data->'json'->>'D_FATURADA_P' as float) end,0) + "
                    + "     coalesce(case when (bd.data->'json'->>'D_P' = '') then null else cast(bd.data->'json'->>'D_P' as float) end,0) as D_FATURADA_P,  "
                    + "     coalesce(case when (bd.data->'json'->>'D_FATURADA_FP' = '') then null else cast(bd.data->'json'->>'D_FATURADA_FP' as float) end,0) + "
                    + "     coalesce(case when (bd.data->'json'->>'D_FP' = '') then null else cast(bd.data->'json'->>'D_FP' as float) end,0) as D_FATURADA_FP, "
                    + "     case when (bd.data->'json'->>'DE_P' = '') then null "
                    + "         else cast(bd.data->'json'->>'DE_P' as float) "
                    + "     end as DE_P, "
                    + "     case when (bd.data->'json'->>'DE_FP' = '') then null "
                    + "         else cast(bd.data->'json'->>'DE_FP' as float) "
                    + "     end as DE_FP, "
                    + "     case when (bd.data->'json'->>'CRE_P' = '') then null "
                    + "         else cast(bd.data->'json'->>'CRE_P' as float) "
                    + "     end as CRE_P, "
                    + "     case when (bd.data->'json'->>'CRE_FP' = '') then null "
                    + "         else cast(bd.data->'json'->>'CRE_FP' as float) "
                    + "     end as CRE_FP, "
                    + "     case when (bd.data->'json'->>'DRE_P' = '') then null "
                    + "         else cast(bd.data->'json'->>'DRE_P' as float) "
                    + "     end as DRE_P, "
                    + "     case when (bd.data->'json'->>'DRE_FP' = '') then null "
                    + "         else cast(bd.data->'json'->>'DRE_FP' as float) "
                    + "     end as DRE_FP, "
                    + "     case when (bd.data->'json'->>'VLR_TOTAL_FATURA' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_TOTAL_FATURA' as float) "
                    + "     end as VLR_TOTAL_FATURA, "
                    + "     case when (bd.data->'json'->>'VLR_BANDEIRA' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_BANDEIRA' as float) "
                    + "     end as VLR_BANDEIRA, "
                    + "     case when (bd.data->'json'->>'VLR_ICMS' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_ICMS' as float) "
                    + "     end as VLR_ICMS, "
                    + "     case when (bd.data->'json'->>'VLR_PIS' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_PIS' as float) "
                    + "     end as VLR_PIS, "
                    + "     case when (bd.data->'json'->>'VLR_COFINS' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_COFINS' as float) "
                    + "     end as VLR_COFINS, "
                    + "     case when (bd.data->'json'->>'VLR_COSIP' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_COSIP' as float) "
                    + "     end as VLR_COSIP, "
                    + "     case when (bd.data->'json'->>'VLR_JUROS' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_JUROS' as float) "
                    + "     end as VLR_JUROS, "  
                    + "     case when (bd.data->'json'->>'VLR_MULTAS' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_MULTAS' as float) "
                    + "     end as VLR_MULTAS, "
                    + "     case when (bd.data->'json'->>'VLR_ATUALIZACAO_MONETARIA' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_ATUALIZACAO_MONETARIA' as float) "
                    + "     end as VLR_ATUALIZACAO_MONETARIA, " 
                    + "     case when (bd.data->'json'->>'VLR_OUTROS_DEBITOS' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_OUTROS_DEBITOS' as float) "
                    + "     end as VLR_OUTROS_DEBITOS, "
                    + "     case when (bd.data->'json'->>'VLR_CREDITOS_DEVOLUCOES' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_CREDITOS_DEVOLUCOES' as float) "
                    + "     end as VLR_CREDITOS_DEVOLUCOES, "
                    + "     case when (bd.data->'json'->>'VLR_TOTAL_LIMINARES' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_TOTAL_LIMINARES' as float) "
                    + "     end as VLR_TOTAL_LIMINARES,"
                    + "     bd.criticalnc, "
                    + "     bd.mediumnc, "
                    + "     bd.lownc, "
                    + "     case when (bd.data->'json'->>'VLR_DE_P' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_DE_P' as float) "
                    + "     end as VLR_DE_P, "
                    + "     case when (bd.data->'json'->>'VLR_DE_FP' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_DE_FP' as float) "
                    + "     end as VLR_DE_FP, "
                    + "     case when (bd.data->'json'->>'VLR_CRE_P' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_CRE_P' as float) "
                    + "     end as VLR_CRE_P, "
                    + "     case when (bd.data->'json'->>'VLR_CRE_FP' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_CRE_FP' as float) "
                    + "     end as VLR_CRE_FP, "
                    + "     case when (bd.data->'json'->>'D_CONTRATADA_FP' = '') then null "
                    + "         else cast(bd.data->'json'->>'D_CONTRATADA_FP' as float) "
                    + "     end AS DEMANDA_CONTRATADA_FP, "
                    + "     case when (bd.data->'json'->>'D_CONTRATADA_P' = '') then null "
                    + "         else cast(bd.data->'json'->>'D_CONTRATADA_P' as float) "
                    + "     end AS DEMANDA_CONTRATADA_P, "
                    + "     EXTRACT (days FROM (bd.period_to - bd.period_from)) AS dias_consumo, "
                    + "     (EXTRACT (days FROM (bd.period_to - bd.period_from)) "
                    + "         - coalesce(cast(bd.data->'json'->>'QTD_DIAS_BAND_VM1' AS float), 0) "
                    + "         - coalesce(cast(bd.data->'json'->>'QTD_DIAS_BAND_VM2' AS float), 0) "
                    + "         - coalesce(cast(bd.data->'json'->>'QTD_DIAS_BAND_AM' AS float), 0)) AS QTD_DIAS_BAND_VD, "
                    + "     case when (bd.data->'json'->>'QTD_DIAS_BAND_AM' = '') then null "
                    + "         else cast(bd.data->'json'->>'QTD_DIAS_BAND_AM' as float) "
                    + "     end as QTD_DIAS_BAND_AM, "
                    + "     case when (bd.data->'json'->>'QTD_DIAS_BAND_VM1' = '') then null "
                    + "         else cast(bd.data->'json'->>'QTD_DIAS_BAND_VM1' as float) "
                    + "     end as QTD_DIAS_BAND_VM1, "
                    + "     case when (bd.data->'json'->>'QTD_DIAS_BAND_VM2' = '') then null "
                    + "         else cast(bd.data->'json'->>'QTD_DIAS_BAND_VM2' as float) "
                    + "     end as QTD_DIAS_BAND_VM2, "
                    + "     case when (bd.data->'json'->>'VLR_DRE_P' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_DRE_P' as float) "
                    + "     end as VLR_DRE_P, "
                    + "     case when (bd.data->'json'->>'VLR_DRE_FP' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_DRE_FP' as float) "
                    + "     end as VLR_DRE_FP, "
                    + "     case when (bd.data->'json'->>'VLR_D_P' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_D_P' as float) "
                    + "     end as VLR_D_P, "
                    + "     case when (bd.data->'json'->>'VLR_D_FP' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_D_FP' as float) "
                    + "     end as VLR_D_FP, "
                    + "     case when (bd.data->'json'->>'VLR_C_P' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_C_P' as float) "
                    + "     end as VLR_C_P, "
                    + "     case when (bd.data->'json'->>'VLR_C_FP' = '') then null "
                    + "         else cast(bd.data->'json'->>'VLR_C_FP' as float) "
                    + "     end as VLR_C_FP "
                    + " from "
                    + "     vero_base_document bd, "
                    + "     vero_contract c, "
                    + "     vero_party_contract pc, "
                    + "     vero_contract_party cp, "
                    + "     vero_contract_template ct, "
                    + "     vero_contract_equipment ce,"
                    + "     vero_equipment eq "
                    + " where "
                    + "     bd.contract_id = c.id "
                    + "     and c.party_contract_id = pc.id "
                    + "     and pc.contract_party_id = cp.id "
                    + "     and pc.contract_template_id = ct.id "
                    + "     and ce.contract_id = c.id "
                    + "     and ce.equipment_id = eq.id "
                    + "     and ct.code like 'aneel%' "
                    + "     and bd.ref_date >= :fromDate "
                    + "     and bd.ref_date <= :toDate "  
                    + condition 
                    + " order by 1, 2, bd.ref_date, bd.period_from ";
		Long fromDate = Long.parseLong(parameters.get(PARAM_FROM_DATE).toString());
		Long toDate = Long.parseLong(parameters.get(PARAM_TO_DATE).toString());
		String companyName = parameters.get(PARAM_COMPANY_ID).toString();
		Long equipmentId = (parameters.get(PARAM_EQUIPMENT_ID) != null)
				? Long.parseLong(parameters.get(PARAM_EQUIPMENT_ID).toString())
				: null;
		List<Object[]> queryResult = getResult(equipmentId, new Date(toDate), new Date(fromDate), companyName);

		if (companyName.equals("%") && equipmentId == 0L) {
			throw new DynamicReportServiceException(
					"Não é possível executar o relatório com os dois campos em branco, favor rever os parâmetros selecionados.");
		}

		if (queryResult.size() > 4000) {
			throw new DynamicReportServiceException(
					"Foram recuperadas muitas faturas de acordo com o filtro indicado, favor rever os parâmetros selecionados.");
		}

		result.put("result", queryResult);

		return result;

	}

	@SuppressWarnings("unchecked")
	private List<Object[]> getResult(Long equipmentId, Date toDate, Date fromDate, String companyName) {

		String condition = "";
		String fqn = "";

		if (equipmentId == 0) {
			if (companyName.equals("Rio Saneamento")) {
				condition = " AND eq.id in (SELECT eq.id FROM vero_equipment eq, vero_metadata vm where "
						+ " eq.id = vm.entity_id and vm.data -> 'json' ->> 'company' like 'Rio Saneamento') ";
			} else {
				condition = " AND eq.id in (SELECT eq.id FROM vero_equipment eq, vero_metadata vm where "
						+ " eq.id = vm.entity_id and vm.data -> 'json' ->> 'company' like 'GAB') ";
			}

		} else {

			Equipment equipment = equipmentDao.findOne(equipmentId);
			fqn = equipment.getNamespace() + "." + equipment.getCode() + "%";
			condition = " and eq.namespace || '.' || eq.code like :fqn ";
		}

		String query = "" + " select " + "     eq.name as eqName, c.external_code, "
				+ "     to_char(bd.ref_date, 'MM/YYYY') as ref_date, "
				+ "     to_char(bd.period_from, 'DD/MM/YYYY') as period_from, "
				+ "     to_char(bd.period_to, 'DD/MM/YYYY') as period_to, " + "     bd.number, "
				+ "     cp.name as cpName, " + "     bd.data->'json'->>'TARIFF_CLASS' as CLASSE_TARIFARIA, "
				+ "     case when bd.data->'json'->>'DT_EMISSAO' = '' then null "
				+ "         else to_char(cast(to_timestamp(cast(bd.data->'json'->>'DT_EMISSAO' as bigint)/1000) as date), 'DD/MM/YYYY') "
				+ "     end as DT_EMISSAO, " + "     case when bd.data->'json'->>'DT_VENCIMENTO' = '' then null "
				+ "         else to_char(cast(to_timestamp(cast(bd.data->'json'->>'DT_VENCIMENTO' as bigint)/1000) as date), 'DD/MM/YYYY') "
				+ "     end as DT_VENCIMENTO, " + "     case when (bd.data->'json'->>'C_P' = '') then null "
				+ "         else cast(bd.data->'json'->>'C_P' as float) " + "     end as C_P, "
				+ "     case when (bd.data->'json'->>'C_FP' = '') then null "
				+ "         else cast(bd.data->'json'->>'C_FP' as float) " + "     end as C_FP, "
				+ "     case when (bd.data->'json'->>'C_TOTAL' = '') then null "
				+ "         else cast(bd.data->'json'->>'C_TOTAL' as float) " + "     end as C_TOTAL, "
				+ "     case when (bd.data->'json'->>'D_MEDIDA_P' = '') then null "
				+ "         else cast(bd.data->'json'->>'D_MEDIDA_P' as float) " + "     end as D_MEDIDA_P, "
				+ "     case when (bd.data->'json'->>'D_MEDIDA_FP' = '') then null "
				+ "         else cast(bd.data->'json'->>'D_MEDIDA_FP' as float) " + "     end as D_MEDIDA_FP, "
				+ "     coalesce(case when (bd.data->'json'->>'D_FATURADA_P' = '') then null else cast(bd.data->'json'->>'D_FATURADA_P' as float) end,0) + "
				+ "     coalesce(case when (bd.data->'json'->>'D_P' = '') then null else cast(bd.data->'json'->>'D_P' as float) end,0) as D_FATURADA_P,  "
				+ "     coalesce(case when (bd.data->'json'->>'D_FATURADA_FP' = '') then null else cast(bd.data->'json'->>'D_FATURADA_FP' as float) end,0) + "
				+ "     coalesce(case when (bd.data->'json'->>'D_FP' = '') then null else cast(bd.data->'json'->>'D_FP' as float) end,0) as D_FATURADA_FP, "
				+ "     case when (bd.data->'json'->>'DE_P' = '') then null "
				+ "         else cast(bd.data->'json'->>'DE_P' as float) " + "     end as DE_P, "
				+ "     case when (bd.data->'json'->>'DE_FP' = '') then null "
				+ "         else cast(bd.data->'json'->>'DE_FP' as float) " + "     end as DE_FP, "
				+ "     case when (bd.data->'json'->>'CRE_P' = '') then null "
				+ "         else cast(bd.data->'json'->>'CRE_P' as float) " + "     end as CRE_P, "
				+ "     case when (bd.data->'json'->>'CRE_FP' = '') then null "
				+ "         else cast(bd.data->'json'->>'CRE_FP' as float) " + "     end as CRE_FP, "
				+ "     case when (bd.data->'json'->>'DRE_P' = '') then null "
				+ "         else cast(bd.data->'json'->>'DRE_P' as float) " + "     end as DRE_P, "
				+ "     case when (bd.data->'json'->>'DRE_FP' = '') then null "
				+ "         else cast(bd.data->'json'->>'DRE_FP' as float) " + "     end as DRE_FP, "
				+ "     case when (bd.data->'json'->>'VLR_TOTAL_FATURA' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_TOTAL_FATURA' as float) "
				+ "     end as VLR_TOTAL_FATURA, " + "     case when (bd.data->'json'->>'VLR_BANDEIRA' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_BANDEIRA' as float) " + "     end as VLR_BANDEIRA, "
				+ "     case when (bd.data->'json'->>'VLR_ICMS' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_ICMS' as float) " + "     end as VLR_ICMS, "
				+ "     case when (bd.data->'json'->>'VLR_PIS' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_PIS' as float) " + "     end as VLR_PIS, "
				+ "     case when (bd.data->'json'->>'VLR_COFINS' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_COFINS' as float) " + "     end as VLR_COFINS, "
				+ "     case when (bd.data->'json'->>'VLR_COSIP' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_COSIP' as float) " + "     end as VLR_COSIP, "
				+ "     case when (bd.data->'json'->>'VLR_JUROS' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_JUROS' as float) " + "     end as VLR_JUROS, "
				+ "     case when (bd.data->'json'->>'VLR_MULTAS' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_MULTAS' as float) " + "     end as VLR_MULTAS, "
				+ "     case when (bd.data->'json'->>'VLR_ATUALIZACAO_MONETARIA' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_ATUALIZACAO_MONETARIA' as float) "
				+ "     end as VLR_ATUALIZACAO_MONETARIA, "
				+ "     case when (bd.data->'json'->>'VLR_OUTROS_DEBITOS' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_OUTROS_DEBITOS' as float) "
				+ "     end as VLR_OUTROS_DEBITOS, "
				+ "     case when (bd.data->'json'->>'VLR_CREDITOS_DEVOLUCOES' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_CREDITOS_DEVOLUCOES' as float) "
				+ "     end as VLR_CREDITOS_DEVOLUCOES, "
				+ "     case when (bd.data->'json'->>'VLR_TOTAL_LIMINARES' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_TOTAL_LIMINARES' as float) "
				+ "     end as VLR_TOTAL_LIMINARES," + "     bd.criticalnc, " + "     bd.mediumnc, " + "     bd.lownc, "
				+ "     case when (bd.data->'json'->>'VLR_DE_P' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_DE_P' as float) " + "     end as VLR_DE_P, "
				+ "     case when (bd.data->'json'->>'VLR_DE_FP' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_DE_FP' as float) " + "     end as VLR_DE_FP, "
				+ "     case when (bd.data->'json'->>'VLR_CRE_P' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_CRE_P' as float) " + "     end as VLR_CRE_P, "
				+ "     case when (bd.data->'json'->>'VLR_CRE_FP' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_CRE_FP' as float) " + "     end as VLR_CRE_FP, "
				+ "     case when (bd.data->'json'->>'D_CONTRATADA_FP' = '') then null "
				+ "         else cast(bd.data->'json'->>'D_CONTRATADA_FP' as float) "
				+ "     end AS DEMANDA_CONTRATADA_FP, "
				+ "     case when (bd.data->'json'->>'D_CONTRATADA_P' = '') then null "
				+ "         else cast(bd.data->'json'->>'D_CONTRATADA_P' as float) "
				+ "     end AS DEMANDA_CONTRATADA_P, "
				+ "     EXTRACT (days FROM (bd.period_to - bd.period_from)) AS dias_consumo, "
				+ "     (EXTRACT (days FROM (bd.period_to - bd.period_from)) "
				+ "         - coalesce(cast(bd.data->'json'->>'QTD_DIAS_BAND_VM1' AS float), 0) "
				+ "         - coalesce(cast(bd.data->'json'->>'QTD_DIAS_BAND_VM2' AS float), 0) "
				+ "         - coalesce(cast(bd.data->'json'->>'QTD_DIAS_BAND_AM' AS float), 0)) AS QTD_DIAS_BAND_VD, "
				+ "     case when (bd.data->'json'->>'QTD_DIAS_BAND_AM' = '') then null "
				+ "         else cast(bd.data->'json'->>'QTD_DIAS_BAND_AM' as float) "
				+ "     end as QTD_DIAS_BAND_AM, "
				+ "     case when (bd.data->'json'->>'QTD_DIAS_BAND_VM1' = '') then null "
				+ "         else cast(bd.data->'json'->>'QTD_DIAS_BAND_VM1' as float) "
				+ "     end as QTD_DIAS_BAND_VM1, "
				+ "     case when (bd.data->'json'->>'QTD_DIAS_BAND_VM2' = '') then null "
				+ "         else cast(bd.data->'json'->>'QTD_DIAS_BAND_VM2' as float) "
				+ "     end as QTD_DIAS_BAND_VM2, " + "     case when (bd.data->'json'->>'VLR_DRE_P' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_DRE_P' as float) " + "     end as VLR_DRE_P, "
				+ "     case when (bd.data->'json'->>'VLR_DRE_FP' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_DRE_FP' as float) " + "     end as VLR_DRE_FP, "
				+ "     case when (bd.data->'json'->>'VLR_D_P' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_D_P' as float) " + "     end as VLR_D_P, "
				+ "     case when (bd.data->'json'->>'VLR_D_FP' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_D_FP' as float) " + "     end as VLR_D_FP, "
				+ "     case when (bd.data->'json'->>'VLR_C_P' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_C_P' as float) " + "     end as VLR_C_P, "
				+ "     case when (bd.data->'json'->>'VLR_C_FP' = '') then null "
				+ "         else cast(bd.data->'json'->>'VLR_C_FP' as float) " + "     end as VLR_C_FP " + " from "
				+ "     vero_base_document bd, " + "     vero_contract c, " + "     vero_party_contract pc, "
				+ "     vero_contract_party cp, " + "     vero_contract_template ct, "
				+ "     vero_contract_equipment ce," + "     vero_equipment eq " + " where "
				+ "     bd.contract_id = c.id " + "     and c.party_contract_id = pc.id "
				+ "     and pc.contract_party_id = cp.id " + "     and pc.contract_template_id = ct.id "
				+ "     and ce.contract_id = c.id " + "     and ce.equipment_id = eq.id "
				+ "     and ct.code like 'aneel%' " + "     and bd.ref_date >= :fromDate "
				+ "     and bd.ref_date <= :toDate " + condition + " order by 1, 2, bd.ref_date, bd.period_from ";

		Map<String, Object> parameters = new HashMap<>();
		parameters.put(PARAM_TO_DATE, toDate);
		parameters.put(PARAM_FROM_DATE, fromDate);
		parameters.put("fqn", fqn);

		return (List<Object[]>) genericDaoService.executeSqlQuery(query, parameters);
	}

        Map<String, Object> parameters = new HashMap<>();
        parameters.put(PARAM_TO_DATE, toDate);  
        parameters.put(PARAM_FROM_DATE, fromDate);  
        parameters.put("fqn", fqn); 
            
        return (List<Object[]>) genericDaoService.executeSqlQuery(query, parameters);
    }
    
}
